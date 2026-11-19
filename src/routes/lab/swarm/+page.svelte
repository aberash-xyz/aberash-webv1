<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import LabNav from '$lib/LabNav.svelte';

	let host;

	const vertexShader = /* glsl */ `
		attribute vec3 aHome;
		uniform float uSize;
		varying float vPush;
		varying float vLight;
		void main() {
			vPush = clamp(distance(position, aHome) * 1.6, 0.0, 1.0);
			vLight = smoothstep(-1.4, 1.4, aHome.y);   // lit from above, like the gap
			vec4 mv = modelViewMatrix * vec4(position, 1.0);
			gl_PointSize = uSize * (1.0 + vPush) / -mv.z;
			gl_Position = projectionMatrix * mv;
		}
	`;

	const fragmentShader = /* glsl */ `
		varying float vPush;
		varying float vLight;
		void main() {
			float a = smoothstep(0.5, 0.0, length(gl_PointCoord - 0.5));
			vec3 violet = vec3(0.39, 0.16, 0.93);
			vec3 cream = vec3(1.0, 0.93, 0.84);
			vec3 col = mix(violet, cream, vLight * 0.55 + vPush * 0.6);
			gl_FragColor = vec4(col, a * (0.55 + vPush * 0.35));
		}
	`;

	onMount(() => {
		const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
		renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
		host.appendChild(renderer.domElement);

		const scene = new THREE.Scene();
		const camera = new THREE.PerspectiveCamera(35, 1, 0.1, 50);
		camera.position.z = 6.4;

		// home positions: torus-knot surface, jittered into the tube so it reads as a swarm
		const knot = new THREE.TorusKnotGeometry(1, 0.32, 1000, 48);
		const home = knot.attributes.position.array.slice();
		knot.dispose();
		for (let i = 0; i < home.length; i++) home[i] += (Math.random() - 0.5) * 0.06;
		const N = home.length / 3;
		const pos = home.slice();
		const vel = new Float32Array(home.length);

		const geometry = new THREE.BufferGeometry();
		geometry.setAttribute('position', new THREE.BufferAttribute(pos, 3).setUsage(THREE.DynamicDrawUsage));
		geometry.setAttribute('aHome', new THREE.BufferAttribute(home, 3));
		const material = new THREE.ShaderMaterial({
			uniforms: { uSize: { value: 0 } },
			vertexShader,
			fragmentShader,
			transparent: true,
			depthWrite: false,
			blending: THREE.AdditiveBlending
		});
		const swarm = new THREE.Points(geometry, material);
		scene.add(swarm);

		// cursor → ray in the swarm's local space; particles near the ray get pushed off it
		const ray = new THREE.Raycaster();
		const local = new THREE.Ray();
		const inv = new THREE.Matrix4();
		const ndc = new THREE.Vector2();
		let inside = false;
		let speed = 0;
		let last = null;

		const onMove = (e) => {
			const r = renderer.domElement.getBoundingClientRect();
			ndc.set(((e.clientX - r.left) / r.width) * 2 - 1, -((e.clientY - r.top) / r.height) * 2 + 1);
			if (last) speed += Math.hypot(e.clientX - last.x, e.clientY - last.y) * 0.004;
			last = { x: e.clientX, y: e.clientY };
			inside = true;
		};
		const onLeave = () => {
			inside = false;
			last = null;
		};

		const size = () => {
			const { clientWidth: w, clientHeight: h } = host;
			renderer.setSize(w, h);
			camera.aspect = w / h;
			camera.updateProjectionMatrix();
			material.uniforms.uSize.value = 0.05 * h * renderer.getPixelRatio();
		};

		const still = matchMedia('(prefers-reduced-motion: reduce)').matches;
		const clock = new THREE.Clock();
		let raf = 0;
		let time = 0;
		const loop = () => {
			raf = requestAnimationFrame(loop);
			const dt = Math.min(clock.getDelta(), 0.05);
			time += dt;
			swarm.rotation.y += dt * (still ? 0.01 : 0.04);
			swarm.rotation.x = Math.sin(time * 0.1) * 0.2;
			swarm.updateMatrixWorld();

			speed *= Math.exp(-dt * 3);
			const push = inside ? 2.5 + Math.min(speed, 1.5) * 6 : 0;
			ray.setFromCamera(ndc, camera);
			local.copy(ray.ray).applyMatrix4(inv.copy(swarm.matrixWorld).invert());
			const { x: ox, y: oy, z: oz } = local.origin;
			const { x: dx, y: dy, z: dz } = local.direction;
			const R = 0.55;
			const spring = 9;
			const damp = Math.exp(-dt * 4.5);

			// ponytail: CPU spring sim, ~49k points is fine; move to a GPGPU texture if we go past ~200k
			for (let i = 0; i < N * 3; i += 3) {
				let px = pos[i], py = pos[i + 1], pz = pos[i + 2];
				let vx = vel[i], vy = vel[i + 1], vz = vel[i + 2];
				if (push) {
					const t = (px - ox) * dx + (py - oy) * dy + (pz - oz) * dz;
					const rx = px - (ox + dx * t), ry = py - (oy + dy * t), rz = pz - (oz + dz * t);
					const d = Math.sqrt(rx * rx + ry * ry + rz * rz);
					if (d < R && d > 1e-5) {
						const f = (1 - d / R) ** 2 * push * dt / d;
						vx += rx * f; vy += ry * f; vz += rz * f;
					}
				}
				// spring home (with a slow breath) and settle
				const b = 1 + Math.sin(time * 0.6 + home[i] * 2.5) * 0.015;
				vx = (vx + (home[i] * b - px) * spring * dt) * damp;
				vy = (vy + (home[i + 1] * b - py) * spring * dt) * damp;
				vz = (vz + (home[i + 2] * b - pz) * spring * dt) * damp;
				pos[i] = px + vx * dt; pos[i + 1] = py + vy * dt; pos[i + 2] = pz + vz * dt;
				vel[i] = vx; vel[i + 1] = vy; vel[i + 2] = vz;
			}
			geometry.attributes.position.needsUpdate = true;
			renderer.render(scene, camera);
		};

		addEventListener('resize', size);
		// touch: finger acts as the cursor while down; lifting it counts as leaving
		const onUp = (e) => e.pointerType !== 'mouse' && onLeave();
		addEventListener('pointermove', onMove);
		addEventListener('pointerdown', onMove);
		addEventListener('pointerup', onUp);
		addEventListener('pointercancel', onUp);
		document.addEventListener('pointerleave', onLeave);
		size();
		loop();

		return () => {
			cancelAnimationFrame(raf);
			removeEventListener('resize', size);
			removeEventListener('pointermove', onMove);
			removeEventListener('pointerdown', onMove);
			removeEventListener('pointerup', onUp);
			removeEventListener('pointercancel', onUp);
			document.removeEventListener('pointerleave', onLeave);
			geometry.dispose();
			material.dispose();
			renderer.dispose();
			renderer.domElement.remove();
		};
	});
</script>

<svelte:head>
	<title>swarm — lab — aberash.</title>
</svelte:head>

<div class="stage" bind:this={host}></div>
<LabNav n={2} name="swarm" />

<style>
	.stage {
		position: fixed;
		inset: 0;
		/* drags go to the lab, not to page scroll/zoom (which would cancel the pointer) */
		touch-action: none;
	}
	.stage :global(canvas) {
		display: block;
	}
</style>
