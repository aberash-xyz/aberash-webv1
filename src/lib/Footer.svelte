<script>
	import { onMount } from 'svelte';
	import { stagger, animate } from 'animejs';
	let year = new Date().getFullYear();

	onMount(() => {
		// re-read on the client — a cached SSR response can outlive the year it was rendered in
		year = new Date().getFullYear();

		animate('.footer__item', {
			opacity: [0, 1],
			duration: 750,
			delay: stagger(150, { start: 1200 })
		});
	});
</script>

<footer class="footer">
	<a
		href="https://docs.google.com/forms/d/14tRWW6ITtClapziqQcGMrS5fNtZjVp3pZ0z_qWG1SJ0/"
		class="footer__item link"><span class="hover-block" />(contact)</a
	>
	<p class="footer__item">c. {year}</p>
</footer>

<style>
	.footer {
		display: flex;
		align-items: center;
		justify-content: space-between;
		color: var(--text);
		width: 100%;
		box-sizing: border-box;
		padding: 20px;
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
		z-index: 10;
	}

	.footer__item {
		font-family: var(--font-mono);
		font-size: 1.4rem;
		letter-spacing: 0.02em;
		line-height: 1.5;
		text-decoration: none;
		color: inherit;
		margin-right: var(--margin);
		opacity: 0;
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
		.footer__item {
			font-size: 1.6rem;
		}
	}

	@media (max-width: 577px) {
		.footer__item {
			font-size: 1.6rem;
		}
	}
</style>
