<script type="module">
	import MainText from '../lib/MainText.svelte';
	import { Canvas } from '@threlte/core';
	import Scene from '$lib/Scene.svelte';
	import BloomPass from '$lib/BloomPass.svelte';
</script>

<div class="container">
	<!-- <MainText /> -->
	<div class="title-container">
		<h1 class="title">aberash</h1>
	</div>
	<div class="image-container">
		<img
			class="background"
			src="https://unsplash.com/photos/the-sun-is-shining-over-the-ocean-waves-1lGzssPAzYU"
			alt={'background image of beach'}
		/>
		<!-- <img class="mask" src="./frame.svg" alt="mask-frame" /> -->
		<svg class="mask" viewBox="0 0 1472 846" fill="none" xmlns="http://www.w3.org/2000/svg">
			<mask id="myMask">
				<rect x="1" y="1" width="380" height="844" stroke="#F8E4DE" stroke-width="2" fill="white" />
				<rect
					x="546"
					y="1"
					width="380"
					height="844"
					stroke="#F8E4DE"
					stroke-width="2"
					fill="white"
				/>
				<rect
					x="1091"
					y="1"
					width="380"
					height="844"
					stroke="#F8E4DE"
					stroke-width="2"
					fill="white"
				/>
			</mask>
			<mask id="path-4-inside-1_324_9" fill="white">
				<path d="M382 0H545V846H382V0Z" />
			</mask>
			<path d="M382 0H545V846H382V0Z" fill="#0D2818" />
			<path
				d="M382 1H545V-1H382V1ZM545 845H382V847H545V845Z"
				fill="#0D2818"
				mask="url(#path-4-inside-1_324_9)"
			/>
			<mask id="path-6-inside-2_324_9" fill="white">
				<path d="M927 0H1090V846H927V0Z" />
			</mask>
			<path d="M927 0H1090V846H927V0Z" fill="#0D2818" />
			<path
				d="M927 1H1090V-1H927V1ZM1090 845H927V847H1090V845Z"
				fill="#0D2818"
				mask="url(#path-6-inside-2_324_9)"
			/>

			<image
				xlink:href="https://images.unsplash.com/photo-1708806016593-e6be239f9d83?q=80&w=1742&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
				x="0"
				y="0"
				width="1472"
				height="846"
				mask="url(#myMask)"
			/>
		</svg>
	</div>
</div>
<div class="postamble">
	<div style="flex: 1;" />
	<p class="trailing-text">holistic creativity</p>
	<a
		href="https://docs.google.com/forms/d/14tRWW6ITtClapziqQcGMrS5fNtZjVp3pZ0z_qWG1SJ0/"
		class="trailing-text"
	>
		contact
	</a>
</div>

<style>
	.stage {
		z-index: 0;
		position: absolute;
	}
	.title-container {
		position: absolute;
		z-index: 2;
		width: 110vw;
		border: solid 0px var(--text);
		border-top-width: 2px;
		border-bottom-width: 2px;
		display: flex;
		align-items: center;
		justify-content: center;
		mix-blend-mode: lighten;
	}
	.title {
		color: var(--text);
		font-size: 50rem;
		font-weight: 500;
		margin: 0;
		line-height: 1.1;
		pointer-events: none;
	}
	.container {
		height: auto;
		width: 100vw;
		aspect-ratio: 173/100;
		display: flex;
		align-items: center;
		justify-content: center;
		overflow: hidden;
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
	}
	.image-container {
		width: calc(100% - 4rem);
		height: 100%;
		aspect-ratio: 173/100;
		overflow: hidden;
		margin: 0 2rem;
		position: relative;
	}
	.background {
		position: absolute;
		height: 96%;
		width: 100%;
		object-fit: cover;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		display: none;
	}
	.mask {
		position: absolute;
		height: 100%;
		width: 100%;
		object-fit: contain;
		overflow: hidden;
	}
	.postamble {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin: 0 2rem;
	}
	.trailing-text {
		color: var(--text);
		text-decoration: none;
		flex: 1;
		text-align: center;
		font-size: 1.8rem;
		line-height: 1.5;
	}
</style>
