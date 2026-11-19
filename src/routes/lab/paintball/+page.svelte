<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
	import { DecalGeometry } from 'three/examples/jsm/geometries/DecalGeometry.js';
	import LabNav from '$lib/LabNav.svelte';

	let host;

	onMount(() => {
		const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
		renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
		host.appendChild(renderer.domElement);

		const scene = new THREE.Scene();
		const camera = new THREE.PerspectiveCamera(35, 1, 0.1, 60);
		camera.position.set(0, 1.6, 7);
		camera.lookAt(0, 0, -4.5); // slightly above, looking down onto her
		camera.updateMatrixWorld();

		// lit like the gap: cream key from above, violet fill from below
		scene.add(new THREE.HemisphereLight(0xfff0dc, 0x2a1466, 0.9));
		const key = new THREE.DirectionalLight(0xfff0dc, 2.2);
		key.position.set(1.5, 4, 3);
		scene.add(key);
		const rim = new THREE.DirectionalLight(0x8a5cff, 1.4);
		rim.position.set(-3, -1, -2);
		scene.add(rim);

		// --- target ---------------------------------------------------------
		let suzanne = null;
		new GLTFLoader().load('/lab/suzanne.glb', (gltf) => {
			gltf.scene.traverse((o) => {
				if (o.isMesh && !suzanne) suzanne = o;
			});
			suzanne.removeFromParent();
			suzanne.material = new THREE.MeshStandardMaterial({ color: 0xe9e4da, roughness: 0.55 });
			suzanne.position.set(0, 0, -4.5);
			suzanne.rotation.set(0, 0, 0);
			scene.add(suzanne);
		});

		// --- gun (primitives), bottom-right, aims at the cursor ---------------
		const gun = new THREE.Group();
		const dark = new THREE.MeshStandardMaterial({ color: 0x1a1726, roughness: 0.4, metalness: 0.3 });
		const body = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.32, 0.9), dark);
		const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.06, 0.07, 0.9, 20), dark);
		barrel.rotation.x = Math.PI / 2;
		barrel.position.z = 0.8;
		const grip = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.42, 0.2), dark);
		grip.position.set(0, -0.3, -0.25);
		grip.rotation.x = 0.25;
		// hopper shows the colour of the next ball
		const hopperMat = new THREE.MeshStandardMaterial({ roughness: 0.25, emissiveIntensity: 0.35 });
		const hopper = new THREE.Mesh(new THREE.SphereGeometry(0.13, 24, 16), hopperMat);
		hopper.position.set(0, 0.25, 0.05);
		const muzzle = new THREE.Object3D();
		muzzle.position.z = 1.25; // lookAt points +z at the target
		const kick = new THREE.Group(); // recoil moves this inside the aimed group
		kick.add(body, barrel, grip, hopper, muzzle);
		gun.add(kick);
		gun.scale.setScalar(0.4);
		scene.add(gun);

		// --- splats ----------------------------------------------------------
		const loader = new THREE.TextureLoader();
		const splats = [1, 2, 3].map((i) => {
			const t = loader.load(`/lab/splat-${i}.svg`);
			t.colorSpace = THREE.SRGBColorSpace;
			return t;
		});
		const decals = [];
		let painted = 0;
		const MAX_DECALS = 160; // ponytail: oldest splat is dropped past this; merge into a paint texture if we want unlimited

		const paint = (hit, color) => {
			const n = hit.face.normal.clone().transformDirection(suzanne.matrixWorld);
			const aim = new THREE.Object3D();
			aim.position.copy(hit.point);
			aim.lookAt(hit.point.clone().add(n));
			aim.rotation.z = Math.random() * Math.PI * 2;
			const s = 0.35 + Math.random() * 0.35;
			const geo = new DecalGeometry(suzanne, hit.point, aim.rotation, new THREE.Vector3(s, s, s));
			// keep the splat stuck to suzanne as she sways
			geo.applyMatrix4(suzanne.matrixWorld.clone().invert());
			const mat = new THREE.MeshStandardMaterial({
				map: splats[(Math.random() * splats.length) | 0],
				color,
				roughness: 0.2,
				transparent: true,
				depthWrite: false,
				polygonOffset: true,
				polygonOffsetFactor: -4,
				polygonOffsetUnits: -4
			});
			const decal = new THREE.Mesh(geo, mat);
			// fixed draw order: newer paint always on top (distance sorting flipped overlaps as she turned)
			decal.renderOrder = ++painted;
			suzanne.add(decal);
			decals.push(decal);
			if (decals.length > MAX_DECALS) {
				const old = decals.shift();
				old.removeFromParent();
				old.geometry.dispose();
				old.material.dispose();
			}
		};

		// --- aiming + firing -------------------------------------------------
		const ray = new THREE.Raycaster();
		const ndc = new THREE.Vector2(0, 0);
		const target = new THREE.Vector3();
		const plane = new THREE.Plane(new THREE.Vector3(0, 0, 1), 4.5); // z = -4.5, suzanne's depth: misses aim past her
		let hue = Math.random();
		const nextColor = () => new THREE.Color().setHSL(hue, 0.85, 0.58);
		const refreshHopper = () => {
			hopperMat.color.copy(nextColor());
			hopperMat.emissive.copy(nextColor());
		};
		refreshHopper();

		const aimAt = () => {
			ray.setFromCamera(ndc, camera);
			const hits = suzanne ? ray.intersectObject(suzanne, false) : [];
			if (hits.length) target.copy(hits[0].point);
			else ray.ray.intersectPlane(plane, target);
		};

		const balls = [];
		const ballGeo = new THREE.SphereGeometry(0.07, 16, 12);
		let recoil = 0;
		const GRAVITY = 4;

		const fire = () => {
			aimAt();
			const color = nextColor();
			const ball = new THREE.Mesh(ballGeo, new THREE.MeshStandardMaterial({ color, emissive: color, emissiveIntensity: 0.3, roughness: 0.2 }));
			muzzle.getWorldPosition(ball.position);
			const speed = 16;
			const dir = target.clone().sub(ball.position).normalize();
			// lift the shot a touch so gravity drops it onto the cursor
			const dist = target.distanceTo(ball.position);
			const t = dist / speed;
			const vel = dir.multiplyScalar(speed);
			vel.y += 0.5 * GRAVITY * t;
			ball.userData = { vel, color, life: 3 };
			scene.add(ball);
			balls.push(ball);
			recoil = 1;
			hue = (hue + 0.618034) % 1; // golden-ratio hue steps: every shot a new, distinct colour
			refreshHopper();
		};

		const toNdc = (e) => {
			const r = renderer.domElement.getBoundingClientRect();
			ndc.set(((e.clientX - r.left) / r.width) * 2 - 1, -((e.clientY - r.top) / r.height) * 2 + 1);
		};
		const onMove = (e) => toNdc(e);
		const onDown = (e) => {
			if (e.target !== renderer.domElement) return; // let the nav links work
			toNdc(e);
			fire();
		};

		const size = () => {
			const { clientWidth: w, clientHeight: h } = host;
			renderer.setSize(w, h);
			camera.aspect = w / h;
			camera.updateProjectionMatrix();
			// park the gun in the bottom-right corner of the view, whatever the aspect
			const v = new THREE.Vector3(0.62, -0.62, 0.5).unproject(camera);
			const d = v.sub(camera.position).normalize();
			gun.position.copy(camera.position).add(d.multiplyScalar(3.2));
		};

		const still = matchMedia('(prefers-reduced-motion: reduce)').matches;
		const clock = new THREE.Clock();
		const prev = new THREE.Vector3();
		const step = new THREE.Vector3();
		let time = 0;
		let raf = 0;
		const loop = () => {
			raf = requestAnimationFrame(loop);
			const dt = Math.min(clock.getDelta(), 0.05);
			time += dt;

			if (suzanne && !still) {
				suzanne.rotation.y = Math.sin(time * 0.4) * 0.5;
				suzanne.rotation.x = Math.sin(time * 0.27) * 0.12;
			}
			suzanne?.updateMatrixWorld();

			aimAt();
			gun.lookAt(target);
			recoil *= Math.exp(-dt * 14);
			kick.position.z = -recoil * 0.18;
			kick.rotation.x = -recoil * 0.12;
			hopper.rotation.y += dt;

			for (let i = balls.length - 1; i >= 0; i--) {
				const b = balls[i];
				const { vel } = b.userData;
				prev.copy(b.position);
				vel.y -= GRAVITY * dt;
				b.position.addScaledVector(vel, dt);
				b.userData.life -= dt;

				// swept hit test along this frame's path, so fast balls can't tunnel through
				let hit = null;
				if (suzanne) {
					step.subVectors(b.position, prev);
					ray.set(prev, step.clone().normalize());
					ray.far = step.length();
					hit = ray.intersectObject(suzanne, false)[0];
					ray.far = Infinity;
				}
				if (hit) paint(hit, b.userData.color);
				if (hit || b.userData.life <= 0) {
					b.removeFromParent();
					b.material.dispose();
					balls.splice(i, 1);
				}
			}
			renderer.render(scene, camera);
		};

		addEventListener('resize', size);
		addEventListener('pointermove', onMove);
		addEventListener('pointerdown', onDown);
		size();
		loop();

		return () => {
			cancelAnimationFrame(raf);
			removeEventListener('resize', size);
			removeEventListener('pointermove', onMove);
			removeEventListener('pointerdown', onDown);
			scene.traverse((o) => {
				o.geometry?.dispose();
				o.material?.dispose();
			});
			splats.forEach((t) => t.dispose());
			renderer.dispose();
			renderer.domElement.remove();
		};
	});
</script>

<svelte:head>
	<title>paintball — lab — aberash.</title>
</svelte:head>

<div class="stage" bind:this={host}></div>
<LabNav n={3} name="paintball" />

<style>
	.stage {
		position: fixed;
		inset: 0;
		touch-action: none;
		cursor: crosshair;
	}
	.stage :global(canvas) {
		display: block;
	}
</style>
