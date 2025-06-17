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
	<div class="header">
		<span class="header__item">40°42′51″ N / 74°00′21″ W</span>
		<span class="header__item">{time}</span>
	</div>

	<h1 class="title">aberash.</h1>

	<h3 class="subtitle">finding the beauty inbetween</h3>

	<div class="video-window">
		<video
			class="video"
			src="dreamy_flowers.mp4"
			poster="hero.jpg"
			playbackRate="0.2"
			muted
			autoplay
			loop
		/>
	</div>

	<h3 class="subtitle">and building things that matter</h3>

	<footer class="footer">
		<div>
			<a
				href="https://docs.google.com/forms/d/14tRWW6ITtClapziqQcGMrS5fNtZjVp3pZ0z_qWG1SJ0/"
				class="footer__item">contact</a
			>
			<a href="https://nostalgiabox.app" class="footer__item">nostalgiabox</a>
			<a href="https://football-village.co" class="footer__item">football-village</a>
		</div>

		<p class="footer__item">c. 2025</p>
	</footer>
	<!-- <MainText /> -->
	<!-- <MatterDemo /> -->
	<!-- <PixiDemo /> -->
	<!-- <PixiMatter /> -->
</div>

<style>
	.container {
		font-size: 62.5%;
		overflow: hidden;
		margin: var(--margin);
		color: var(--secondary);
		position: relative;
		height: auto;
		/* height: calc(100vh - var(--margin) * 2); */
	}
	.title {
		font-size: 20rem;
		color: var(--secondary);
		font-weight: 100;
		text-align: center;
		margin: 2.5rem 0 0rem 0;
	}
	.subtitle {
		font-size: 3.2rem;
		text-align: center;
		font-weight: 100;
		margin: 2rem 0;
	}
	.header,
	.footer {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}
	.header__item {
		/* font: 2rem/1.5; */
		font-size: 2rem;
		line-height: 1.5;
		padding: 0.4rem 0.8rem;
	}
	.video-window {
		aspect-ratio: 5/6;
		overflow: hidden;
		width: 80%;
		height: auto;
		max-width: 550px;
		min-width: 300px;
		border-radius: 25rem 25rem 1rem 1rem;
		justify-self: center;
		margin-bottom: 8rem;
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0px 5px 10px rgba(0, 0, 0, 0.25);
	}
	.video {
		height: 100%;
		width: 100%;
		object-position: center;
		object-fit: cover;
	}

	.hero {
		aspect-ratio: 16/9;
		width: 100%;
		border-radius: 3rem;
	}
	.footer {
		width: calc(100%);
	}
	.footer__item {
		font-size: 2rem;
		line-height: 1.5;
		text-decoration: none;
		color: inherit;
		margin-right: var(--margin);
	}

	@media (max-width: 1200px) {
		.title {
			font-size: 24rem;
		}
		.subtitle {
			font-size: 3.2rem;
		}
		.header__item,
		.footer__item {
			font-size: 1.8rem;
		}
	}

	@media (max-width: 800px) {
		.title {
			font-size: 18rem;
		}
		.subtitle {
			font-size: 2.4rem;
		}
		.header__item,
		.footer__item {
			font-size: 1.6rem;
		}
	}

	@media (max-width: 577px) {
		.title {
			font-size: 11rem;
		}
		.subtitle {
			font-size: 2rem;
		}
		.header__item,
		.footer__item {
			font-size: 1.6rem;
		}
	}
</style>
