<script>
	import { onMount } from 'svelte';
	import Matter from 'matter-js';
	import { Application, Assets, Texture, Sprite, Text, TextStyle } from 'pixi.js';

	onMount(async () => {
		const stage = document.getElementById('pixi');
		const app = new Application();
		await app.init({
			background: '#0d2818',
			resizeTo: window
		});
		// Assets.addBundle('fonts', [{ alias: 'Soria', src: '/fonts/soria-font.woff' }]);
		// await Assets.loadBundle('fonts');

		// const text1 = new Text({
		// 	text: 'aberash',
		// 	style: { fontFamily: 'Soria', fontSize: 300, fill: '#f8e4de' }
		// });
		const bunnyTexture = await Assets.load('circle-sprite.png');
		const sprite = new Sprite(bunnyTexture);

		// app.stage.addChild(text1);
		app.stage.addChild(sprite);
		stage.appendChild(app.canvas);
	});
</script>

<div id="pixi" />

<style>
</style>
