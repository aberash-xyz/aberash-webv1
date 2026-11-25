<script>
	// Stable fluids (Jos Stam, 1999) on the GPU, after mofu-dev's write-up:
	// https://mofu-dev.com/en/blog/stable-fluids/
	//
	// Every frame:
	//   1. advect   — carry velocity along itself (semi-Lagrangian: look backwards, sample)
	//   2. force    — splat the pointer's motion into the velocity field
	//   3. project  — make the field divergence-free (incompressible):
	//                 divergence → solve pressure (Jacobi) → subtract pressure gradient
	//   4. dye      — carry colour along the final velocity; that's what you see
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import LabHint from '$lib/LabHint.svelte';
	import LabNav from '$lib/LabNav.svelte';

	let host;

	const vertexShader = /* glsl */ `
		varying vec2 vUv;
		void main() { vUv = uv; gl_Position = vec4(position.xy, 0.0, 1.0); }
	`;

	// read the field where this pixel's fluid *came from* — unconditionally stable for any dt
	const advect = /* glsl */ `
		varying vec2 vUv;
		uniform sampler2D uVel, uSrc;
		uniform vec2 uTexel;       // one sim cell in uv (velocity is stored in cells/second)
		uniform float uDt, uKeep;  // uKeep: fraction left after this step (fades everything out)
		void main() {
			vec2 from = vUv - uDt * texture2D(uVel, vUv).xy * uTexel;
			gl_FragColor = uKeep * texture2D(uSrc, from);
		}
	`;

	// gaussian dab of `uValue` around the pointer
	const splat = /* glsl */ `
		varying vec2 vUv;
		uniform sampler2D uSrc;
		uniform vec2 uPoint;
		uniform vec3 uValue;
		uniform float uRadius, uAspect;
		void main() {
			vec2 d = vUv - uPoint;
			d.x *= uAspect;
			gl_FragColor = vec4(texture2D(uSrc, vUv).xyz + uValue * exp(-dot(d, d) / uRadius), 1.0);
		}
	`;

	// how much fluid leaves each cell (central differences)
	const divergence = /* glsl */ `
		varying vec2 vUv;
		uniform sampler2D uVel;
		uniform vec2 uTexel;
		void main() {
			float l = texture2D(uVel, vUv - vec2(uTexel.x, 0.0)).x;
			float r = texture2D(uVel, vUv + vec2(uTexel.x, 0.0)).x;
			float b = texture2D(uVel, vUv - vec2(0.0, uTexel.y)).y;
			float t = texture2D(uVel, vUv + vec2(0.0, uTexel.y)).y;
			gl_FragColor = vec4(0.5 * (r - l + t - b), 0.0, 0.0, 1.0);
		}
	`;

	// one Jacobi iteration of the Poisson equation  ∇²p = ∇·u
	const pressure = /* glsl */ `
		varying vec2 vUv;
		uniform sampler2D uP, uDiv;
		uniform vec2 uTexel;
		void main() {
			float l = texture2D(uP, vUv - vec2(uTexel.x, 0.0)).x;
			float r = texture2D(uP, vUv + vec2(uTexel.x, 0.0)).x;
			float b = texture2D(uP, vUv - vec2(0.0, uTexel.y)).x;
			float t = texture2D(uP, vUv + vec2(0.0, uTexel.y)).x;
			gl_FragColor = vec4((l + r + b + t - texture2D(uDiv, vUv).x) * 0.25, 0.0, 0.0, 1.0);
		}
	`;

	// u − ∇p : what's left has no divergence, so it swirls instead of piling up
	const gradient = /* glsl */ `
		varying vec2 vUv;
		uniform sampler2D uVel, uP;
		uniform vec2 uTexel;
		void main() {
			float l = texture2D(uP, vUv - vec2(uTexel.x, 0.0)).x;
			float r = texture2D(uP, vUv + vec2(uTexel.x, 0.0)).x;
			float b = texture2D(uP, vUv - vec2(0.0, uTexel.y)).x;
			float t = texture2D(uP, vUv + vec2(0.0, uTexel.y)).x;
			vec2 v = texture2D(uVel, vUv).xy - 0.5 * vec2(r - l, t - b);
			gl_FragColor = vec4(v, 0.0, 1.0);
		}
	`;

	const display = /* glsl */ `
		varying vec2 vUv;
		uniform sampler2D uDye;
		void main() {
			vec3 c = texture2D(uDye, vUv).rgb;
			c = c / (1.0 + c);                      // soft tonemap so overlaps don't clip to white
			gl_FragColor = vec4(vec3(0.012, 0.016, 0.055) + c, 1.0); // #03040e underneath
		}
	`;

	onMount(() => {
		const renderer = new THREE.WebGLRenderer({ antialias: false });
		renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
		renderer.autoClear = false;
		host.appendChild(renderer.domElement);

		const scene = new THREE.Scene();
		const camera = new THREE.Camera();
		const quad = new THREE.Mesh(new THREE.PlaneGeometry(2, 2));
		scene.add(quad);

		const pass = (fragmentShader, uniforms) =>
			new THREE.ShaderMaterial({ vertexShader, fragmentShader, uniforms, depthTest: false, depthWrite: false });
		const mats = {
			advect: pass(advect, { uVel: { value: null }, uSrc: { value: null }, uTexel: { value: new THREE.Vector2() }, uDt: { value: 0 }, uKeep: { value: 1 } }),
			splat: pass(splat, { uSrc: { value: null }, uPoint: { value: new THREE.Vector2() }, uValue: { value: new THREE.Vector3() }, uRadius: { value: 0 }, uAspect: { value: 1 } }),
			divergence: pass(divergence, { uVel: { value: null }, uTexel: { value: new THREE.Vector2() } }),
			pressure: pass(pressure, { uP: { value: null }, uDiv: { value: null }, uTexel: { value: new THREE.Vector2() } }),
			gradient: pass(gradient, { uVel: { value: null }, uP: { value: null }, uTexel: { value: new THREE.Vector2() } }),
			display: pass(display, { uDye: { value: null } })
		};

		const run = (mat, values, target) => {
			for (const k in values) mat.uniforms[k].value = values[k];
			quad.material = mat;
			renderer.setRenderTarget(target);
			renderer.render(scene, camera);
		};

		// read from one, write to the other, swap
		const pair = (w, h) => {
			const make = () =>
				new THREE.WebGLRenderTarget(w, h, {
					type: THREE.HalfFloatType,
					minFilter: THREE.LinearFilter,
					magFilter: THREE.LinearFilter,
					wrapS: THREE.ClampToEdgeWrapping,
					wrapT: THREE.ClampToEdgeWrapping,
					depthBuffer: false
				});
			const p = { read: make(), write: make() };
			p.swap = () => ([p.read, p.write] = [p.write, p.read]);
			p.dispose = () => (p.read.dispose(), p.write.dispose());
			return p;
		};

		let vel, prs, dye, div, texel, aspect;
		const size = () => {
			const { clientWidth: w, clientHeight: h } = host;
			renderer.setSize(w, h);
			aspect = w / h;
			// ponytail: fixed sim size; raise SIM for finer swirls at the cost of fill rate
			const SIM = 160;
			const DYE = 720;
			const sw = Math.round(aspect >= 1 ? SIM * aspect : SIM);
			const sh = Math.round(aspect >= 1 ? SIM : SIM / aspect);
			[vel, prs, dye, div].forEach((t) => t?.dispose());
			vel = pair(sw, sh);
			prs = pair(sw, sh);
			dye = pair(Math.round(aspect >= 1 ? DYE * aspect : DYE), Math.round(aspect >= 1 ? DYE : DYE / aspect));
			div = new THREE.WebGLRenderTarget(sw, sh, { type: THREE.HalfFloatType, depthBuffer: false });
			texel = new THREE.Vector2(1 / sw, 1 / sh);
		};

		// pointer → force + dye
		const pending = [];
		let last = null;
		let hue = Math.random();
		const color = new THREE.Color();
		const onMove = (e) => {
			const x = e.clientX / innerWidth;
			const y = 1 - e.clientY / innerHeight;
			if (last) {
				const dx = (x - last.x) / texel.x; // motion in sim cells this event
				const dy = (y - last.y) / texel.y;
				if (dx || dy) pending.push({ x, y, dx, dy });
			}
			last = { x, y };
		};
		const onDown = (e) => {
			last = null;
			onMove(e);
		};
		const onUp = () => (last = null);

		const push = ({ x, y, dx, dy }, dt) => {
			hue = (hue + 0.002) % 1;
			color.setHSL(0.68 + Math.sin(hue * Math.PI * 2) * 0.12, 0.85, 0.55); // violet ↔ magenta ↔ blue
			const r = 0.0018;
			run(mats.splat, { uSrc: vel.read.texture, uPoint: new THREE.Vector2(x, y), uValue: new THREE.Vector3(dx / dt, dy / dt, 0).multiplyScalar(0.6), uRadius: r, uAspect: aspect }, vel.write);
			vel.swap();
			const strength = Math.min(Math.hypot(dx, dy) * 0.08, 0.6);
			run(mats.splat, { uSrc: dye.read.texture, uValue: new THREE.Vector3(color.r, color.g, color.b).multiplyScalar(strength), uRadius: r * 1.4 }, dye.write);
			dye.swap();
		};

		const still = matchMedia('(prefers-reduced-motion: reduce)').matches;
		const clock = new THREE.Clock();
		let raf = 0;
		const loop = () => {
			raf = requestAnimationFrame(loop);
			const dt = Math.min(clock.getDelta(), 1 / 30);

			// 1. advect velocity
			run(mats.advect, { uVel: vel.read.texture, uSrc: vel.read.texture, uTexel: texel, uDt: dt, uKeep: Math.exp(-dt * 0.8) }, vel.write);
			vel.swap();

			// 2. forces
			while (pending.length) push(pending.shift(), Math.max(dt, 1 / 120));

			// 3. projection
			run(mats.divergence, { uVel: vel.read.texture, uTexel: texel }, div);
			for (let i = 0; i < 24; i++) {
				run(mats.pressure, { uP: prs.read.texture, uDiv: div.texture, uTexel: texel }, prs.write);
				prs.swap();
			}
			run(mats.gradient, { uVel: vel.read.texture, uP: prs.read.texture, uTexel: texel }, vel.write);
			vel.swap();

			// 4. dye rides the velocity; fades slower than motion so trails linger
			run(mats.advect, { uVel: vel.read.texture, uSrc: dye.read.texture, uTexel: texel, uDt: dt, uKeep: Math.exp(-dt * (still ? 1.5 : 0.45)) }, dye.write);
			dye.swap();

			run(mats.display, { uDye: dye.read.texture }, null);
		};

		addEventListener('resize', size);
		addEventListener('pointermove', onMove);
		addEventListener('pointerdown', onDown);
		addEventListener('pointerup', onUp);
		addEventListener('pointercancel', onUp);
		size();

		// a few swirls so the page isn't empty before you touch it
		for (let i = 0; i < 6; i++) {
			const a = Math.random() * Math.PI * 2;
			pending.push({ x: 0.3 + Math.random() * 0.4, y: 0.3 + Math.random() * 0.4, dx: Math.cos(a) * 7, dy: Math.sin(a) * 7 });
		}
		loop();

		return () => {
			cancelAnimationFrame(raf);
			removeEventListener('resize', size);
			removeEventListener('pointermove', onMove);
			removeEventListener('pointerdown', onDown);
			removeEventListener('pointerup', onUp);
			removeEventListener('pointercancel', onUp);
			[vel, prs, dye, div].forEach((t) => t.dispose());
			Object.values(mats).forEach((m) => m.dispose());
			quad.geometry.dispose();
			renderer.dispose();
			renderer.domElement.remove();
		};
	});
</script>

<svelte:head>
	<title>fluid — lab — aberash.</title>
</svelte:head>

<div class="stage" bind:this={host}></div>
<!-- 5s: dye is ~90% faded by then -->
<LabHint />
<LabNav n={4} name="fluid" />
<a class="credit" href="https://mofu-dev.com/en/blog/stable-fluids/" target="_blank" rel="noopener">after mofu-dev — stable fluids ↗</a>

<style>
	.stage {
		position: fixed;
		inset: 0;
		touch-action: none;
	}
	.stage :global(canvas) {
		display: block;
	}
	.credit {
		position: fixed;
		left: 50%;
		bottom: 10.4rem;
		transform: translateX(-50%);
		font-family: var(--font-mono);
		font-size: 1.1rem;
		color: rgba(242, 240, 234, 0.35);
		text-decoration: none;
		white-space: nowrap;
		z-index: 1;
		transition: color 0.3s ease;
	}
	.credit:hover {
		color: var(--text);
	}
</style>
