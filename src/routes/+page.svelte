<script type="module">
	import { animate, splitText, createTimeline, stagger } from 'animejs';
	import FloatingBoxes from '$lib/FloatingBoxes.svelte';
	import { onMount } from 'svelte';
	let time = '00:00';

	onMount(() => {
		const { words, chars } = splitText('.title', {
			words: { wrap: 'clip' },
			chars: true
		});
		// Immediately hide the chars - no flash!
		chars.forEach((char) => (char.style.opacity = 0));

		const { words: subtitleWords } = splitText('.subtitle', { words: { wrap: 'clip' } });
		subtitleWords.forEach((word) => (word.style.opacity = 0));

		const timeline = createTimeline({
			defaults: { ease: 'inOut(3)', duration: 750 }
		});

		timeline.add([chars, subtitleWords], {
			y: [{ to: '100%' }],
			duration: 0
		});

		timeline.add(['.title', '.subtitle'], {
			opacity: {
				from: 0,
				to: 1
			},
			duration: 0
		});

		// Title animation
		timeline.add(chars, {
			y: [{ to: ['100%', '0%'] }],
			opacity: {
				from: 0,
				to: 1
			},
			duration: 510,
			ease: 'out(3)',
			delay: stagger(17, { from: 'random', start: 300 })
		});

		timeline.add(
			subtitleWords,
			{
				y: [{ to: ['100%', '0%'] }],
				opacity: {
					from: 0,
					to: 1
				},
				duration: 400,
				ease: 'out(3)',
				delay: stagger(103)
			},
			'-=100'
		);

		// Links fade in after subtitle
		timeline.add(
			'.links-container>.link',
			{
				opacity: [0, 1],
				duration: 600,
				delay: stagger(150)
			},
			'-=100'
		);
		timeline.add('.decorative-box', {
			opacity: {
				from: 0,
				to: 1
			},
			delay: stagger(300, { start: 500 })
		});
	});
</script>

<FloatingBoxes />

<div class="container">
	<div class="main-center">
		<h1 class="title">aberash<span class="title__period">.</span></h1>
		<h3 class="subtitle">finding the beauty inbetween <br /> and building things that matter</h3>
		<div class="links-container">
			<a class="link" href="/mood"><span class="hover-block" />(mood)</a>
			<a class="link" href="https://instagram.com/aberash.studio"
				><span class="hover-block" />(@aberash.studio)</a
			>
		</div>
	</div>

	<div class="decorative-box box-1"></div>
	<div class="decorative-box box-2"></div>
	<div class="decorative-box box-3"></div>
</div>

<style>
	.container {
		font-size: 62.5%;
		overflow: hidden;
		color: var(--secondary);
		position: relative;
		height: 100vh;
		display: grid;
		grid-template-columns: repeat(10, 1fr);
		grid-template-rows: repeat(8, 1fr);
		box-sizing: border-box;
		z-index: 10;
		pointer-events: none;
	}

	.container * {
		pointer-events: auto;
	}
	.main-center {
		display: grid;
		grid-column: 3/9;
		grid-row: 3/7;
		align-self: center;
		justify-self: center;
		gap: var(--extra-large);
	}
	.title {
		font-size: 10rem;
		color: var(--secondary);
		font-weight: 600;
		letter-spacing: -1.4px;
		text-align: center;
		opacity: 0;
	}
	.title__period {
		display: inline;
		transform-origin: center center;
	}
	.subtitle {
		font-size: 2rem;
		font-style: italic;
		text-align: center;
		font-weight: 200;
		opacity: 0;
	}
	.links-container {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: var(--large);
		max-width: 400px;
		margin: auto;
	}
	.link {
		display: inline-block;
		position: relative;
		color: inherit;
		font-size: 1.6rem;
		text-decoration: none;
		padding: 0.4rem 0.8rem;
		overflow: hidden;
		font-weight: 200;
		text-align: center;
		opacity: 0;
	}
	.hover-block {
		height: 100%;
		width: 100%;
		background: var(--hover-color);
		position: absolute;
		left: -100%;
		transition: ease-in-out 0.3s;
	}
	.decorative-box {
		height: 106px;
		width: 133px;
		background: var(--text);
		opacity: 0;
	}
	.box-1 {
		transform: scale(0.25);
		grid-row: 2/2;
		grid-column: 4/4;
	}
	.box-2 {
		transform: scale(0.5);
		grid-row: 7/7;
		grid-column: 2/2;
	}
	.box-3 {
		grid-row: 6/6;
		grid-column: 9/9;
	}
	.link:hover .hover-block {
		left: 100%;
	}

	@media (max-width: 1200px) {
		.title {
			font-size: 10rem;
		}
		.subtitle {
			font-size: 1.8rem;
			font-style: italic;
		}
	}

	@media (max-width: 800px) {
		.container {
			grid-template-columns: repeat(5, 1fr);
		}
		.main-center {
			grid-column: 2/6;
		}
		.title {
			font-size: 9rem;
		}
		.subtitle {
			font-size: 1.8rem;
		}
	}

	@media (max-width: 577px) {
		.container {
			grid-template-columns: repeat(3, 1fr);
		}
		.title {
			font-size: 6rem;
		}
		.main-center {
			align-self: none;
			justify-self: none;
			grid-column: 2/4;
		}
		.box-2 {
			grid-column: 1/1;
			transform: scale(0.4);
		}
		.box-3 {
			grid-column: 4/4;
			transform: scale(0.8);
			display: none;
		}
		.subtitle {
			font-size: 1.8rem;
			font-style: italic;
		}
	}
	@keyframes pulse {
		0% {
			transform: scale(1);
			opacity: 1;
		}
		50% {
			opacity: 0;
			transform: scale(1.1);
		}
		100% {
			transform: scale(1);
		}
	}
	@keyframes blink {
		0% {
			opacity: 1;
		}
		25% {
			opacity: 0;
		}
		50% {
			opacity: 0;
		}
		100% {
			opacity: 1;
		}
	}
</style>
