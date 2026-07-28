<script type="module">
	import { animate, splitText, createTimeline, stagger } from 'animejs';
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
	});
</script>

<div class="container">
	<div class="main-center">
		<h1 class="title">aberash<span class="title__period">.</span></h1>
		<h3 class="subtitle">finding the beauty inbetween</h3>
		<div class="links-container">
			<a class="link" href="/mood"><span class="hover-block" />(mood)</a>
			<span class="link link--pending">(lab-coming.soon)</span>
		</div>
	</div>
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
		font-family: var(--font-display);
		font-size: 14rem;
		color: var(--secondary);
		font-weight: 400;
		font-style: italic;
		letter-spacing: 0;
		line-height: 0.9;
		text-align: center;
		opacity: 0;
	}
	.title__period {
		display: inline;
		transform-origin: center center;
	}
	.subtitle {
		font-family: var(--font-mono);
		font-size: 1.5rem;
		font-style: normal;
		text-align: center;
		font-weight: 400;
		letter-spacing: 0.02em;
		line-height: 1.6;
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
		font-family: var(--font-mono);
		font-size: 1.4rem;
		text-decoration: none;
		padding: 0.4rem 0.8rem;
		overflow: hidden;
		font-weight: 400;
		letter-spacing: 0.02em;
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
	.link:hover .hover-block {
		left: 100%;
	}
	/* placeholder until the lab exists — not clickable */
	.link--pending {
		cursor: default;
		opacity: 0;
	}

	@media (max-width: 1200px) {
		.title {
			font-size: 14rem;
		}
		.subtitle {
			font-size: 1.4rem;
		}
	}

	@media (max-width: 800px) {
		.container {
			grid-template-columns: repeat(5, 1fr);
		}
		.main-center {
			grid-column: 1/6;
			justify-self: stretch;
			padding: 0 var(--margin);
			box-sizing: border-box;
		}
		.title {
			font-size: 12rem;
		}
		.subtitle {
			font-size: 1.3rem;
		}
	}

	@media (max-width: 577px) {
		.container {
			grid-template-columns: repeat(3, 1fr);
		}
		.title {
			font-size: 8.5rem;
		}
		.main-center {
			grid-column: 1/4;
			justify-self: stretch;
			padding: 0 var(--margin);
			box-sizing: border-box;
		}
		.subtitle {
			font-size: 1.3rem;
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
