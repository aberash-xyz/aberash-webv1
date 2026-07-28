<script>
	import { onMount } from 'svelte';
	import { stagger, animate } from 'animejs';
	import { fade, fly } from 'svelte/transition';
	import { cubicIn, cubicOut } from 'svelte/easing';

	let mobileMenuOpen = false;
	let year = new Date().getFullYear();

	const toggleMenu = () => {
		mobileMenuOpen = !mobileMenuOpen;
	};

	const closeMenu = () => {
		mobileMenuOpen = false;
	};

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

<!-- Desktop Footer -->
<footer class="footer desktop-footer">
	<a
		href="https://docs.google.com/forms/d/14tRWW6ITtClapziqQcGMrS5fNtZjVp3pZ0z_qWG1SJ0/"
		class="footer__item link"><span class="hover-block" />(contact)</a
	>
	<a href="https://nostalgiabox.app" class="footer__item link"
		><span class="hover-block" />(nostalgiabox)</a
	>
	<a href="https://instagram.com/aberash.studio" class="footer__item link"
		><span class="hover-block" />(@aberash.studio)</a
	>
	<p class="footer__item">c. {year}</p>
</footer>

<!-- Mobile Hamburger / Close Button -->
<button
	class="mobile-menu-toggle {mobileMenuOpen ? 'open' : ''}"
	on:click={toggleMenu}
	aria-label="Toggle menu"
>
	<span class="hamburger-line line-1"></span>
	<span class="hamburger-line line-2"></span>
	<span class="hamburger-line line-3"></span>
</button>

<!-- Mobile Full-Page Nav -->
{#if mobileMenuOpen}
	<div
		class="mobile-nav-overlay"
		on:click={closeMenu}
		in:fade={{ duration: 300 }}
		out:fade={{ duration: 650 }}
	>
		<nav
			class="mobile-nav"
			on:click|stopPropagation
			in:fly={{ x: -1000, duration: 400, easing: cubicOut }}
			out:fly={{ x: -1000, duration: 600, easing: cubicIn }}
		>
			<div class="mobile-nav-links">
				<a
					href="https://docs.google.com/forms/d/14tRWW6ITtClapziqQcGMrS5fNtZjVp3pZ0z_qWG1SJ0/"
					class="mobile-nav-link"
					on:click={closeMenu}
				>
					(contact)
				</a>
				<a href="https://nostalgiabox.app" class="mobile-nav-link" on:click={closeMenu}>
					(nostalgiabox)
				</a>
				<a
					href="https://instagram.com/aberash.studio"
					class="mobile-nav-link"
					on:click={closeMenu}
				>
					(@aberash.studio)
				</a>
				<p class="mobile-nav-link">c. {year}</p>
			</div>
		</nav>
	</div>
{/if}

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

	/* Mobile Menu Toggle */
	.mobile-menu-toggle {
		display: none;
		position: fixed;
		bottom: 20px;
		left: 20px;
		z-index: 3000;
		background: transparent;
		border: none;
		cursor: pointer;
		padding: 10px;
		flex-direction: column;
		gap: 6px;
		width: 50px;
		height: 50px;
		justify-content: center;
		align-items: center;
	}

	.hamburger-line {
		width: 30px;
		height: 2px;
		background-color: var(--text);
		transition: all 0.3s ease;
		display: block;
		position: absolute;
	}

	.line-1 {
		transform: translateY(-8px);
	}

	.line-2 {
		transform: translateY(0);
	}

	.line-3 {
		transform: translateY(8px);
	}

	/* Animate to X when open */
	.mobile-menu-toggle.open .line-1 {
		transform: rotate(45deg);
		background-color: var(--text);
	}

	.mobile-menu-toggle.open .line-2 {
		opacity: 0;
	}

	.mobile-menu-toggle.open .line-3 {
		transform: rotate(-45deg);
		background-color: var(--text);
	}

	/* Mobile Navigation Overlay */
	.mobile-nav-overlay {
		display: none;
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		background: rgba(0, 0, 0, 0.5);
		z-index: 2000;
	}

	.mobile-nav {
		position: relative;
		width: 100%;
		height: 100%;
		background: #0e1f4a;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.mobile-nav-links {
		display: flex;
		flex-direction: column;
		gap: 3rem;
		align-items: center;
	}

	.mobile-nav-link {
		font-family: var(--font-mono);
		font-size: 2.2rem;
		color: var(--text);
		text-decoration: none;
		padding: 1rem 2rem;
	}

	/* Show mobile elements on small screens */
	@media (max-width: 800px) {
		.desktop-footer {
			display: none;
		}

		.mobile-menu-toggle {
			display: flex;
		}

		.mobile-nav-overlay {
			display: block;
		}
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
