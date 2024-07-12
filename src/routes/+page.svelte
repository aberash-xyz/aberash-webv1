<script type="module">
	import { onMount } from 'svelte';
	let time = '00:00';

	onMount(() => {
		setTimeout(() => {
			updateTime();
		}, 1000);
	});

	const updateTime = () => {
		const date = new Date();
		const minutes = date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes();
		time = date.getHours() + ':' + minutes;
	};
</script>

<div class="container">
	<div class="header">
		<span class="header__item">Doing it</span>
		<span class="header__item">40°42′51″ N / 74°00′21″ W</span>
		<span class="header__item">{time}</span>
	</div>

	<h1 class="title">aberash.</h1>

	<h3 class="subtitle">finding the beauty inbetween</h3>

	<img class="hero" src="hero.jpg" alt="field of flowers shot on film" />

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

		<p class="footer__item">c. 2024</p>
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
	}
	.title {
		font-size: 30rem;
		color: var(--secondary);
		font-weight: 100;
		text-align: center;
		margin: 2.5rem 0 8rem 0;
	}
	.subtitle {
		font-size: 4rem;
		text-align: center;
		font-weight: 100;
		margin: 6rem 0;
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
	}
	.hero {
		aspect-ratio: 16/9;
		width: 100%;
		border-radius: 3rem;
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
