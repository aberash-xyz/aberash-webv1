<script>
	import Bulb from './Bulb.svelte';
	import { T, useThrelte, useTask, useLoader } from '@threlte/core';
	import { onMount } from 'svelte';
	import { OrbitControls } from '@threlte/extras';
	import { TextureLoader } from 'three';
	import { Pane, Slider } from 'svelte-tweakpane-ui';
	let spotlight;
	let pointLightY = 0;
	let lightX = -1.52;
	let lightY = 0.11;
	let lightZ = 0.33;
	let luminanceThreshold = 0.9;

	const { load } = useLoader(TextureLoader);
	import {
		EffectComposer,
		EffectPass,
		RenderPass,
		SMAAEffect,
		SMAAPreset,
		BloomEffect,
		KernelSize
	} from 'postprocessing';

	const { scene, renderer, camera, size } = useThrelte();

	// Adapt the WEBGL Renderer https://github.com/pmndrs/postprocessing#usage
	const composer = new EffectComposer(renderer);

	const setupEffectComposer = (camera) => {
		composer.removeAllPasses();
		composer.addPass(new RenderPass(scene, camera));
		composer.addPass(
			new EffectPass(
				camera,
				new BloomEffect({
					intensity: 5,
					luminanceThreshold: luminanceThreshold,
					height: 1024,
					width: 1024,
					luminanceSmoothing: 0.08,
					mipmapBlur: false,
					kernelSize: KernelSize.MEDIUM
				})
			)
		);
		composer.addPass(
			new EffectPass(
				camera,
				new SMAAEffect({
					preset: SMAAPreset.LOW
				})
			)
		);
	};
	// We need to set up the passes according to the camera in use
	$: setupEffectComposer($camera);
	$: composer.setSize($size.width, $size.height);
	const { renderStage, autoRender } = useThrelte();
	// We need to disable auto rendering as soon as this component is
	// mounted and restore the previous state when it is unmounted.
	onMount(() => {
		let before = autoRender.current;
		autoRender.set(false);
		return () => autoRender.set(before);
	});
	useTask(
		(delta) => {
			composer.render(delta);
		},
		{ stage: renderStage, autoInvalidate: false }
	);
</script>

<T.PerspectiveCamera
	makeDefault
	position={[0.0, 0.0, 0.135]}
	on:create={({ ref }) => {
		ref.lookAt(0, 0, 0);
	}}
	fov={50}
>
	<OrbitControls enableDamping />
</T.PerspectiveCamera>
<Bulb />

<T.DirectionalLight position={[lightX, lightY, lightZ]} intensity={1} color={'white'} />
<T.DirectionalLight position={[2.5, -1.15, 0.75]} intensity={1} color={'white'} />
<T.PointLight position={[0.01, 0, 0]} color="white" intensity={0.05} decay={1} />
<!-- <T.PointLight position={[0.01, 0 - pointLightY, 0]} color="#dd6e0f" intensity={3} decay={1} /> -->

<!-- <T.Mesh position={[0, -0.5, -2.75]}>
	<T.PlaneGeometry args={[5, 3.33]} />
	{#await load('textures/planeMap.png') then map}
		<T.MeshBasicMaterial {map} />
	{/await}
</T.Mesh> -->

<!-- TwEAKE PANE -->

<Pane position="fixed" title="controls">
	<Slider label={'point light Y'} min={-5} max={0} bind:value={pointLightY} />
	<Slider label={'lightX'} max={5} min={-5} bind:value={lightX} />
	<Slider label={'lightY'} max={5} min={-5} bind:value={lightY} />
	<Slider label={'lightZ'} max={5} min={-5} bind:value={lightZ} />
	<Slider label={'luminanceThreshold'} max={5} min={0} bind:value={luminanceThreshold} />
</Pane>
