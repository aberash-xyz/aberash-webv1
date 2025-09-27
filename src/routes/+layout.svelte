<script>
	import '../global.css';

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

<div class="header">
	<span class="header__item">40°42′51″ N / 74°00′21″ W</span>
	<span class="header__item">{time}</span>
</div>
<slot />
<footer class="footer">
	<div>
		<a
			href="https://docs.google.com/forms/d/14tRWW6ITtClapziqQcGMrS5fNtZjVp3pZ0z_qWG1SJ0/"
			class="footer__item link"><span class="hover-block" />contact</a
		>
		<a href="https://nostalgiabox.app" class="footer__item link"
			><span class="hover-block" />nostalgiabox</a
		>
		<a href="https://football-village.co" class="footer__item link"
			><span class="hover-block" />football-village</a
		>
	</div>

	<p class="footer__item">c. 2025</p>
</footer>

<style>
	.header,
	.footer {
		display: flex;
		align-items: center;
		justify-content: space-between;
		color: var(--text);
		width: 100vw;
	}
	.footer {
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
		padding: 0 8px;
	}
	.header__item {
		font-size: 1.6rem;
		line-height: 1.5;
		padding: 0.4rem 0.8rem;
		min-width: 50px;
	}
	.footer {
		width: calc(100%);
	}
	.footer__item {
		font-size: 1.6rem;
		letter-spacing: -0.3px;
		line-height: 1.5;
		text-decoration: none;
		color: inherit;
		margin-right: var(--margin);
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
		.header__item,
		.footer__item {
			font-size: 1.6rem;
		}
	}

	@media (max-width: 800px) {
		.header__item,
		.footer__item {
			font-size: 1.6rem;
		}
	}

	@media (max-width: 577px) {
		.header__item,
		.footer__item {
			font-size: 1.6rem;
		}
	}
</style>
