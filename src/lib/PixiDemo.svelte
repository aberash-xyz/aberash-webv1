<script>
	import { onMount } from 'svelte';
	import { Application, Assets, Texture, Sprite } from 'pixi.js';

	onMount(async () => {
		const stage = document.getElementById('pixi');
		const app = new Application();
		await app.init({
			background: '#0d2818',
			resizeTo: window
		});
		const bunnyTexture = await Assets.load('circle-sprite.png');
		const sprite = new Sprite(bunnyTexture);

		app.stage.addChild(sprite);
		stage.appendChild(app.canvas);
	});
</script>

<div id="pixi" />

<style>
</style>
