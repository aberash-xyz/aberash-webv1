<script>
	const labs = [
		{ title: 'blob', kind: 'shader', year: '2026', href: '/lab/blob', a: '#6320ee', b: '#120a2e' },
		{ title: 'swarm', kind: 'particles', year: '2026', href: '/lab/swarm', a: '#fff0dc', b: '#3b1d8f' },
		{ title: 'paintball', kind: 'interaction', year: '2026', href: '/lab/paintball', a: '#ff5a8a', b: '#2a1466' },
		{ title: 'fluid', kind: 'simulation', year: '2026', href: '/lab/fluid', a: '#9b5cff', b: '#03040e' }
	];

	// figma v2 uneven grid on 4 columns: [2 1 1] [1 1 2] [3 1], repeating
	const spans = [2, 1, 1, 1, 1, 2, 3, 1];
	$: cells = spans.map((span, i) => ({ span, i, lab: labs[i] }));
</script>

<svelte:head>
	<title>lab — aberash.</title>
</svelte:head>

<section class="lab">
	<div class="head">
		<a class="home" href="/"><span class="hover-block"></span>(home)</a>
		<span>lab — {String(labs.length).padStart(2, '0')}</span>
	</div>

	<div class="grid">
		{#each cells as { span, i, lab }}
			{#if lab}
				<a class="cell" href={lab.href} style="--span:{span}; --a:{lab.a}; --b:{lab.b}">
					<div class="card">
						<div class="face front">
							<span class="meta tl">{String(i + 1).padStart(2, '0')}</span>
							<span class="meta tr">{lab.year}</span>
							<span class="title">{lab.title}</span>
							<span class="meta bl">{lab.kind}</span>
						</div>
						<div class="face back">
							<span class="meta bl">{lab.title}</span>
							<span class="meta br">view →</span>
						</div>
					</div>
				</a>
			{:else}
				<div class="cell empty" style="--span:{span}">
					<span class="meta tl">{String(i + 1).padStart(2, '0')}</span>
				</div>
			{/if}
		{/each}
	</div>
</section>

<style>
	.lab {
		padding: 7.2rem var(--margin) 10rem;
		color: var(--text);
	}
	.head {
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-family: var(--font-mono);
		font-size: 1.3rem;
		padding: 0 0 1.6rem;
	}
	.head > span {
		opacity: 0.6;
	}

	/* same (home) link as /mood: violet block wipes across on hover */
	.home {
		position: relative;
		isolation: isolate; /* keeps the z-index:-1 block behind the text, not behind the page */
		overflow: hidden;
		padding: 0.4rem 0.8rem;
		color: inherit;
		text-decoration: none;
	}
	.hover-block {
		position: absolute;
		top: 0;
		left: -100%;
		width: 100%;
		height: 100%;
		background: var(--hover-color);
		transition: left 0.3s ease-in-out;
		z-index: -1;
	}
	.home:hover .hover-block {
		left: 100%;
	}

	.grid {
		display: grid;
		grid-template-columns: repeat(4, 1fr);
		grid-auto-rows: 31rem 21rem 31rem;
		border-top: 1px solid rgba(242, 240, 234, 0.1);
		border-left: 1px solid rgba(242, 240, 234, 0.1);
	}
	.cell {
		grid-column: span var(--span);
		position: relative;
		border-right: 1px solid rgba(242, 240, 234, 0.1);
		border-bottom: 1px solid rgba(242, 240, 234, 0.1);
		perspective: 1600px;
		color: inherit;
		text-decoration: none;
	}

	/* hover flips the card on Y to reveal the visual */
	.card {
		position: absolute;
		inset: 0;
		transform-style: preserve-3d;
		transition: transform 1.6s cubic-bezier(0.19, 1, 0.22, 1);
	}
	.cell:hover .card,
	.cell:focus-visible .card {
		transform: rotateY(180deg);
	}
	.face {
		position: absolute;
		inset: 0;
		display: grid;
		place-items: center;
		backface-visibility: hidden;
		background: var(--primary);
	}
	.back {
		transform: rotateY(180deg);
		background:
			radial-gradient(circle at 50% 45%, rgba(255, 255, 255, 0.2), transparent 60%),
			linear-gradient(var(--a), var(--b));
	}

	.title {
		font-family: var(--font-display);
		font-style: italic;
		font-size: calc(clamp(4rem, 4.4vw, 6.4rem) * (1 + (var(--span) - 1) * 0.6));
		line-height: 0.9;
	}
	.meta {
		position: absolute;
		font-family: var(--font-mono);
		font-size: 1.1rem;
		letter-spacing: 0.04em;
		opacity: 0.5;
	}
	.back .meta {
		opacity: 1;
	}
	.tl { top: 1.2rem; left: 1.2rem; }
	.tr { top: 1.2rem; right: 1.2rem; }
	.bl { bottom: 1.2rem; left: 1.2rem; }
	.br { bottom: 1.2rem; right: 1.2rem; }
	.empty .meta {
		opacity: 0.2;
	}

	@media (max-width: 800px) {
		.grid {
			grid-template-columns: repeat(2, 1fr);
			grid-auto-rows: 24rem;
		}
		.cell {
			grid-column: span min(var(--span), 2);
		}
	}
	@media (prefers-reduced-motion: reduce) {
		.card {
			transition-duration: 0.01s;
		}
	}
</style>
