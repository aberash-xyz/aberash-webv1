<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import LabNav from '$lib/LabNav.svelte';

	let host;

	// ashima simplex noise (MIT), 3d
	const noise = /* glsl */ `
		vec3 mod289(vec3 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }
		vec4 mod289(vec4 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }
		vec4 permute(vec4 x) { return mod289(((x * 34.0) + 1.0) * x); }
		vec4 taylorInvSqrt(vec4 r) { return 1.79284291400159 - 0.85373472095314 * r; }
		float snoise(vec3 v) {
			const vec2 C = vec2(1.0 / 6.0, 1.0 / 3.0);
			const vec4 D = vec4(0.0, 0.5, 1.0, 2.0);
			vec3 i = floor(v + dot(v, C.yyy));
			vec3 x0 = v - i + dot(i, C.xxx);
			vec3 g = step(x0.yzx, x0.xyz);
			vec3 l = 1.0 - g;
			vec3 i1 = min(g.xyz, l.zxy);
			vec3 i2 = max(g.xyz, l.zxy);
			vec3 x1 = x0 - i1 + C.xxx;
			vec3 x2 = x0 - i2 + C.yyy;
			vec3 x3 = x0 - D.yyy;
			i = mod289(i);
			vec4 p = permute(permute(permute(
				i.z + vec4(0.0, i1.z, i2.z, 1.0))
				+ i.y + vec4(0.0, i1.y, i2.y, 1.0))
				+ i.x + vec4(0.0, i1.x, i2.x, 1.0));
			float n_ = 0.142857142857;
			vec3 ns = n_ * D.wyz - D.xzx;
			vec4 j = p - 49.0 * floor(p * ns.z * ns.z);
			vec4 x_ = floor(j * ns.z);
			vec4 y_ = floor(j - 7.0 * x_);
			vec4 x = x_ * ns.x + ns.yyyy;
			vec4 y = y_ * ns.x + ns.yyyy;
			vec4 h = 1.0 - abs(x) - abs(y);
			vec4 b0 = vec4(x.xy, y.xy);
			vec4 b1 = vec4(x.zw, y.zw);
			vec4 s0 = floor(b0) * 2.0 + 1.0;
			vec4 s1 = floor(b1) * 2.0 + 1.0;
			vec4 sh = -step(h, vec4(0.0));
			vec4 a0 = b0.xzyw + s0.xzyw * sh.xxyy;
			vec4 a1 = b1.xzyw + s1.xzyw * sh.zzww;
			vec3 p0 = vec3(a0.xy, h.x);
			vec3 p1 = vec3(a0.zw, h.y);
			vec3 p2 = vec3(a1.xy, h.z);
			vec3 p3 = vec3(a1.zw, h.w);
			vec4 norm = taylorInvSqrt(vec4(dot(p0, p0), dot(p1, p1), dot(p2, p2), dot(p3, p3)));
			p0 *= norm.x; p1 *= norm.y; p2 *= norm.z; p3 *= norm.w;
			vec4 m = max(0.6 - vec4(dot(x0, x0), dot(x1, x1), dot(x2, x2), dot(x3, x3)), 0.0);
			m = m * m;
			return 42.0 * dot(m * m, vec4(dot(p0, x0), dot(p1, x1), dot(p2, x2), dot(p3, x3)));
		}
	`;

	const vertexShader = /* glsl */ `
		uniform float uTime;
		uniform vec3 uMouse;   // point on the unit sphere under the cursor
		uniform float uHover;  // 0..1+, eased; spikes with cursor speed
		varying vec3 vNormal;
		varying vec3 vView;
		varying float vPull;
		${noise}

		// slow breathing noise + a bulge (with a ripple) toward the cursor
		vec3 displace(vec3 dir) {
			float n = snoise(dir * 1.3 + uTime * 0.15) * 0.16 + snoise(dir * 3.1 - uTime * 0.24) * 0.04;
			float d = distance(dir, uMouse);
			float m = exp(-d * d * 5.0) * uHover;
			float ripple = sin(d * 16.0 - uTime * 5.0) * 0.025 * m;
			return dir * (1.0 + n + m * 0.32 + ripple);
		}

		void main() {
			vec3 dir = normalize(position);
			vec3 p = displace(dir);

			// normal from two displaced neighbours along the surface
			vec3 t = normalize(abs(dir.y) > 0.99 ? cross(dir, vec3(1.0, 0.0, 0.0)) : cross(dir, vec3(0.0, 1.0, 0.0)));
			vec3 b = cross(dir, t);
			float e = 0.01;
			vec3 pt = displace(normalize(dir + t * e));
			vec3 pb = displace(normalize(dir + b * e));
			vNormal = normalize(normalMatrix * cross(pt - p, pb - p));

			float d = distance(dir, uMouse);
			vPull = exp(-d * d * 5.0) * uHover;

			vec4 mv = modelViewMatrix * vec4(p, 1.0);
			vView = normalize(-mv.xyz);
			gl_Position = projectionMatrix * mv;
		}
	`;

	// lit like the gap: one cream key light from above, violet in the shadows, soft rim
	const fragmentShader = /* glsl */ `
		varying vec3 vNormal;
		varying vec3 vView;
		varying float vPull;

		void main() {
			vec3 N = normalize(vNormal);
			vec3 V = normalize(vView);
			vec3 L = normalize(vec3(0.25, 1.0, 0.45));
			vec3 cream = vec3(1.0, 0.93, 0.84);
			vec3 deep = vec3(0.07, 0.04, 0.2);
			vec3 violet = vec3(0.39, 0.13, 0.93);

			float diff = max(dot(N, L), 0.0);
			float wrap = max((dot(N, L) + 0.5) / 1.5, 0.0);   // soft, skin-like falloff
			float spec = pow(max(dot(reflect(-L, N), V), 0.0), 36.0);
			float rim = pow(1.0 - max(dot(N, V), 0.0), 2.6);

			vec3 col = mix(deep, violet, wrap * 0.75);
			col = mix(col, cream * 0.9, diff * diff * 0.35);
			col += cream * spec * 0.55;
			col += vec3(0.75, 0.6, 1.0) * rim * 0.6;
			col += cream * vPull * 0.12;   // warms up where it reaches for you
			gl_FragColor = vec4(col, 1.0);
		}
	`;

	onMount(() => {
		const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
		renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
		host.appendChild(renderer.domElement);

		const scene = new THREE.Scene();
		const camera = new THREE.PerspectiveCamera(35, 1, 0.1, 50);
		camera.position.z = 5.2;

		const uniforms = {
			uTime: { value: 0 },
			uMouse: { value: new THREE.Vector3(0, 0, 1) },
			uHover: { value: 0 }
		};
		// detail 128 ≈ 165k vertices: smooth enough for the ripple
		const geometry = new THREE.IcosahedronGeometry(1, 128);
		const material = new THREE.ShaderMaterial({ uniforms, vertexShader, fragmentShader });
		const blob = new THREE.Mesh(geometry, material);
		scene.add(blob);

		// cursor → point on the (undisplaced) unit sphere, in the blob's local space
		const ray = new THREE.Raycaster();
		const sphere = new THREE.Sphere(new THREE.Vector3(), 1.15);
		const ndc = new THREE.Vector2();
		const hit = new THREE.Vector3();
		const target = new THREE.Vector3(0, 0, 1);
		let want = 0;
		let speed = 0;
		let last = null;

		const onMove = (e) => {
			const r = renderer.domElement.getBoundingClientRect();
			ndc.set(((e.clientX - r.left) / r.width) * 2 - 1, -((e.clientY - r.top) / r.height) * 2 + 1);
			if (last) speed += Math.hypot(e.clientX - last.x, e.clientY - last.y) * 0.004;
			last = { x: e.clientX, y: e.clientY };
			ray.setFromCamera(ndc, camera);
			if (ray.ray.intersectSphere(sphere, hit)) {
				target.copy(blob.worldToLocal(hit.clone())).normalize();
				want = 1;
			} else want = 0;
		};
		const onLeave = () => {
			want = 0;
			last = null;
		};

		const size = () => {
			const { clientWidth: w, clientHeight: h } = host;
			renderer.setSize(w, h);
			camera.aspect = w / h;
			camera.updateProjectionMatrix();
		};

		const still = matchMedia('(prefers-reduced-motion: reduce)').matches;
		const clock = new THREE.Clock();
		let raf = 0;
		const loop = () => {
			raf = requestAnimationFrame(loop);
			const dt = Math.min(clock.getDelta(), 0.05);
			uniforms.uTime.value += dt * (still ? 0.25 : 1);
			blob.rotation.y += dt * 0.04;
			// ease everything so it feels like flesh, not a pointer
			uniforms.uMouse.value.lerp(target, 1 - Math.exp(-dt * 6)).normalize();
			speed *= Math.exp(-dt * 3);
			const goal = want * (1 + Math.min(speed, 1.2));
			uniforms.uHover.value += (goal - uniforms.uHover.value) * (1 - Math.exp(-dt * 4));
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
	<title>blob — lab — aberash.</title>
</svelte:head>

<div class="stage" bind:this={host}></div>
<LabNav n={1} name="blob" />

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
