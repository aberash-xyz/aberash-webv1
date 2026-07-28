<script>
	import '../global.css';

	import { createTimeline, stagger, splitText, createAnimatable } from 'animejs';
	import Footer from '$lib/Footer.svelte';
	import GradientBackground from '$lib/GradientBackground.svelte';
	import { onMount } from 'svelte';
	let time = '00:00';
	let animatableSquare;

	onMount(() => {
		animatableSquare = createAnimatable('.shadow', {
			x: 500, // Define the x duration to be 500ms
			y: 500, // Define the y duration to be 500ms
			ease: 'out(3)'
		});
		updateTime();

		// animateGeoCoordinates();

		window.addEventListener('mousemove', onMouseMove);
	});

	const animateGeoCoordinates = () => {
		splitText('.geocoordinates', {
			chars: `<span class="char-3d">
				<em class="face face-top">{value}</em>
				<em class="face face-front">{value}</em>
				<em class="face face-bottom">{value}</em>
			</span>`
		});

		const charsStagger = stagger(100, { start: 0 });

		createTimeline({ defaults: { ease: 'linear', loop: true, duration: 750 } })
			.add('.char-3d', { rotateX: -90 }, charsStagger)
			.add('.char-3d .face-top', { opacity: [0.5, 0] }, charsStagger)
			.add('.char-3d .face-front', { opacity: [1, 0.5] }, charsStagger)
			.add('.char-3d .face-bottom', { opacity: [0.5, 1] }, charsStagger);
	};

	const updateTime = () => {
		const date = new Date();
		const minutes = date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes();
		const seconds = date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds();
		time = date.getHours() + ':' + minutes + ':' + seconds;
		setTimeout(() => {
			updateTime();
		}, 1000);
	};

	const onMouseMove = (e) => {
		animatableSquare.x(e.clientX); // Animate the x value in 500ms
		animatableSquare.y(e.clientY); // Animate the y value in 500ms
	};
</script>

<GradientBackground />

<div class="root">
	<div class="shadow"></div>
	<div class="header">
		<span class="header__item geocoordinates">40°42′51″ N / 74°00′21″ W</span>
		<span class="header__item">{time}</span>
	</div>
	<slot />
	<Footer />
</div>

<style>
	.root {
		width: 100vw;
		position: relative;
		z-index: 2;
	}
	.shadow {
		height: 16px;
		width: 24px;
		background-color: var(--text);
	}
	.header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		color: var(--text);
		width: 100%;
		box-sizing: border-box;
		padding: 20px;
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
	}
	.header__item {
		font-family: var(--font-mono);
		font-size: 1.4rem;
		letter-spacing: 0.02em;
		line-height: 1.5;
		padding: 0.4rem 0.8rem;
		min-width: 50px;
	}

	.char-3d {
		display: inline-block;
		position: relative;
		transform-style: preserve-3d;
		perspective: 1000px;
	}

	.char-3d .face {
		position: absolute;
		top: 0;
		left: 0;
		font-style: normal;
		backface-visibility: hidden;
	}

	.char-3d .face-front {
		position: relative;
	}

	.char-3d .face-top {
		transform: rotateX(90deg) translateZ(0.5em);
		transform-origin: top center;
	}

	.char-3d .face-bottom {
		transform: rotateX(-90deg) translateZ(0.5em);
		transform-origin: bottom center;
	}

	@media (max-width: 1200px) {
		.header__item {
			font-size: 1.4rem;
		}
	}

	@media (max-width: 800px) {
		.header__item {
			font-size: 1.3rem;
		}
	}

	@media (max-width: 577px) {
		.header__item {
			font-size: 1.1rem;
		}
	}
</style>
