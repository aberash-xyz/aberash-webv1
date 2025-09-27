<script type="module">
	import { animate } from 'animejs';
	import { onMount } from 'svelte';
	let time = '00:00';

	onMount(() => {
		updateTime();
		trackMousePosition();
		listenForMouseLeave();
	});

	const trackMousePosition = () => {
		document.addEventListener('mousemove', (event) => {
			const x = event.clientX - window.innerWidth / 2;
			const y = event.clientY - window.innerHeight / 2;

			animate('.video-window', {
				rotateX: -y / (window.innerWidth / 20),
				rotateY: x / (window.innerHeight / 20),
				duration: 550,
				easing: 'easeInOutQuad'
			});
		});
	};

	const listenForMouseLeave = () => {
		document.addEventListener('mouseleave', () => {
			animate('.video-window', {
				rotateX: 0,
				rotateY: 0,
				duration: 550,
				easing: 'easeInOutQuad'
			});
		});
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
</script>

<div class="container">
	<h1 class="title">aberash<span class="title__period">.</span></h1>

	<h3 class="subtitle">finding the beauty inbetween</h3>

	<h3 class="subtitle">and building things that matter</h3>

	<div class="links-container">
		<a class="link" href="/mood"><span class="hover-block" />(mood)</a>
		<a class="link" href="https://instagram.com/aberash.studio"
			><span class="hover-block" />(@aberash.studio)</a
		>
	</div>
</div>

<style>
	.container {
		font-size: 62.5%;
		overflow: hidden;
		margin: var(--margin);
		color: var(--secondary);
		position: relative;
		height: auto;
	}
	.title {
		font-size: 10rem;
		color: var(--secondary);
		font-weight: 500;
		letter-spacing: -1.4px;
		text-align: center;
		margin: 2.5rem 0 0rem 0;
	}
	.title__period {
		display: inline-block;
		transform-origin: center center;
	}
	.subtitle {
		font-size: 1.8rem;
		font-style: italic;
		text-align: center;
		font-weight: 300;
		margin: 2rem 0;
	}
	.links-container {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 8px;
		max-width: 400px;
		margin: auto;
		margin-top: 64px;
	}
	.link {
		display: inline-block;
		position: relative;
		color: inherit;
		font-size: 1.6rem;
		text-decoration: none;
		padding: 0.4rem 0.8rem;
		overflow: hidden;
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

	@media (max-width: 1200px) {
		.title {
			font-size: 10rem;
		}
		.subtitle {
			font-family: 'Source Serif';
			font-size: 1.8rem;
			font-style: italic;
		}
	}

	@media (max-width: 800px) {
		.title {
			font-size: 9rem;
		}
		.subtitle {
			font-family: 'Source Serif';
			font-size: 1.8rem;
		}
	}

	@media (max-width: 577px) {
		.title {
			font-size: 9rem;
		}
		.subtitle {
			font-family: 'Source Serif';
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
