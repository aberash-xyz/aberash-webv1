<script>
	import { onMount } from 'svelte';
	import { animate, stagger, createAnimatable } from 'animejs';

	const word = [...'aberash.'];

	// placeholder projects + gradient visuals until real assets exist
	const projects = [
		{ title: 'nostalgiabox', kind: 'product', year: '2025', href: 'https://nostalgiabox.app', a: '#e9a35b', b: '#3a1c12' },
		{ title: 'synia sports', kind: 'brand / web', year: '2025', href: 'https://syniasports.com', a: '#6320ee', b: '#0b0b2a' },
		{ title: 'mood', kind: 'archive', year: '2024', href: '/mood', a: '#e8c6c0', b: '#6b4a5a' },
		{ title: 'lab', kind: 'experiments', year: '2026', href: '/lab', a: '#c7d36f', b: '#1f3b2d' }
	];

	let hero, photo, text, title, sub, cue;
	let letters = [];
	let lit = word.map(() => false);
	let active = -1;
	let cardTurn = 0;
	let card;

	const clamp = (v) => Math.min(1, Math.max(0, v));
	// 0→1 as p moves from a to b
	const range = (p, a, b) => clamp((p - a) / (b - a));
	const mix = (a, b, t) => a + (b - a) * t;

	onMount(() => {
		let mouseX = -9999;
		let mouseY = -9999;
		let raf = 0;
		const still = matchMedia('(prefers-reduced-motion: reduce)').matches;

		const frame = () => {
			raf = 0;
			const y = scrollY;
			const p = clamp(y / hero.offsetHeight);

			// parallax: photo drifts up at half speed, text a little faster than the page
			if (!still) {
				photo.style.transform = `translate3d(0, ${Math.min(y, hero.offsetHeight) * 0.5}px, 0)`;
				text.style.transform = `translate3d(0, ${-Math.min(y, hero.offsetHeight) * 0.15}px, 0)`;
			}

			// photo fades out before it reaches the list; once gone, drop it (and the text) from
			// compositing entirely — idle GPU layers left a faint half-screen seam on the dark bg
			const fade = 1 - range(p, 0.05, 0.55);
			photo.style.opacity = fade;
			photo.style.visibility = fade ? '' : 'hidden';
			text.style.visibility = p < 0.6 ? '' : 'hidden';

			// title grows, spreads and fades as you leave the hero
			const t = range(p, 0, 0.6);
			title.style.transform = `scale(${mix(1, 1.4, t)})`;
			title.style.letterSpacing = `${mix(0, 0.2, t)}em`;
			title.style.opacity = 1 - range(p, 0.2, 0.6);
			sub.style.opacity = 1 - range(p, 0, 0.3);
			cue.style.opacity = 1 - range(p, 0, 0.1);

			// letters inside the slit of light (screen centre) or near the cursor catch the light
			const beam = innerWidth / 2;
			const reach = mix(Math.max(40, innerWidth * 0.03), innerWidth * 0.2, t);
			lit = letters.map((el) => {
				const b = el.getBoundingClientRect();
				const cx = b.left + b.width / 2;
				const cy = b.top + b.height / 2;
				return Math.abs(cx - beam) < reach || Math.hypot(cx - mouseX, cy - mouseY) < 110;
			});
		};
		const tick = () => (raf ||= requestAnimationFrame(frame));
		const onMove = (e) => {
			mouseX = e.clientX;
			mouseY = e.clientY;
			tick();
			follow?.x(e.clientX);
			follow?.y(e.clientY);
		};

		const follow = createAnimatable(card, { x: 600, y: 600, ease: 'out(3)' });

		addEventListener('scroll', tick, { passive: true });
		addEventListener('resize', tick);
		addEventListener('pointermove', onMove);
		frame();

		animate(letters, {
			opacity: [0, 1],
			y: ['40%', '0%'],
			duration: 900,
			ease: 'out(3)',
			delay: stagger(60, { start: 200, from: 'center' })
		});

		return () => {
			removeEventListener('scroll', tick);
			removeEventListener('resize', tick);
			removeEventListener('pointermove', onMove);
			cancelAnimationFrame(raf);
		};
	});

	// card spins in the direction of travel through the list, like the v1 flip
	const hover = (i) => {
		if (i === active) return;
		cardTurn += i > active ? -180 : 180;
		active = i;
	};
</script>

<section class="hero" bind:this={hero}>
	<div class="photo" bind:this={photo}></div>
	<div class="hero__text" bind:this={text}>
		<h1 class="title" bind:this={title} aria-label="aberash.">
			{#each word as ch, i}
				<span class="letter" class:lit={lit[i]} bind:this={letters[i]} aria-hidden="true">
					<span class="pin">{ch}</span><span class="serif">{ch}</span>
				</span>
			{/each}
		</h1>
		<p class="sub" bind:this={sub}>finding the beauty inbetween</p>
	</div>
	<a class="cue" href="#index" bind:this={cue}>index ↓</a>
</section>

<section class="index" id="index">
	<div class="index__head">
		<span>selected work</span>
		<span>index — {String(projects.length).padStart(2, '0')}</span>
	</div>
	<!-- svelte-ignore a11y-no-static-element-interactions -->
	<ul class="list" class:hovering={active >= 0} on:pointerleave={() => (active = -1)}>
		{#each projects as p, i}
			<li class:active={active === i} on:pointerenter={() => hover(i)}>
				<svelte:element this={p.href ? 'a' : 'span'} href={p.href} class="row">
					<span class="row__i">{String(i + 1).padStart(2, '0')}</span>
					<span class="row__title">{p.title}</span>
					<span class="row__kind">{p.kind}</span>
					<span class="row__year">{p.year}</span>
				</svelte:element>
			</li>
		{/each}
	</ul>
	<div class="spacer"></div>
</section>

<div class="card-anchor" bind:this={card} class:show={active >= 0}>
	<div class="card" style="--ry:{cardTurn}deg; --face:{Math.abs(cardTurn / 180) % 2 ? 180 : 0}deg">
		{#each projects as p, i}
			<div class="card__face" class:on={active === i} style="--a:{p.a}; --b:{p.b}">
				<span>{p.title}</span><span>{p.href ? 'view →' : 'soon'}</span>
			</div>
		{/each}
	</div>
</div>

<style>
	/* ---------- hero (figma v4) ---------- */
	.hero {
		position: relative;
		height: 100vh;
		height: 100svh;
		/* clip sideways only (the growing title), keep the photo's fade spilling down into the list */
		overflow-x: clip;
	}
	.photo {
		position: absolute;
		inset: 0;
		background: url('/gap.webp') center / cover no-repeat;
		/* bottom fade to var(--primary) is baked into gap.webp: a CSS mask on this moving layer
		   composited in GPU tiles that didn't match the page background (visible seams) */
		will-change: transform;
	}
	.hero__text {
		position: absolute;
		inset: 0;
		z-index: 1;
		display: grid;
		place-content: center;
		justify-items: center;
		gap: 2.4rem;
		color: var(--text);
		pointer-events: none;
		will-change: transform;
	}
	.title {
		display: flex;
		font-size: clamp(9rem, 24vw, 36rem);
		line-height: 0.85;
		font-weight: 400;
		will-change: transform;
	}

	/* each letter stacks both faces; light crossfades between them */
	.letter {
		display: inline-grid;
		justify-items: center;
		align-items: baseline;
		opacity: 0;
	}
	.letter > span {
		grid-area: 1 / 1;
		transition:
			opacity 0.45s ease,
			filter 0.45s ease,
			color 0.45s ease;
	}
	.pin {
		font-family: var(--font-display);
		font-style: italic;
		color: rgba(242, 240, 234, 0.24);
	}
	.serif {
		font-family: 'Source Serif', serif;
		font-style: italic;
		font-weight: 300;
		font-size: 0.8em;
		color: #fff6e6;
		text-shadow: 0 0 0.25em rgba(255, 246, 230, 0.55);
		opacity: 0;
		filter: blur(6px);
	}
	.letter.lit .pin {
		opacity: 0;
		filter: blur(6px);
	}
	.letter.lit .serif {
		opacity: 1;
		filter: blur(0);
	}

	.sub {
		margin: 0;
		font-family: var(--font-mono);
		font-size: 1.5rem;
		letter-spacing: 0.02em;
		opacity: 0.7;
	}
	.cue {
		position: absolute;
		z-index: 1;
		left: 50%;
		bottom: 8rem;
		transform: translateX(-50%);
		font-family: var(--font-mono);
		font-size: 1.3rem;
		color: var(--text);
		text-decoration: none;
		opacity: 0.7;
	}

	/* ---------- index ---------- */
	.index {
		position: relative;
		z-index: 1;
		color: var(--text);
		padding: 4rem var(--margin) 0;
	}
	.index__head {
		display: flex;
		justify-content: space-between;
		font-family: var(--font-mono);
		font-size: 1.3rem;
		padding: 0 0.8rem 1.6rem;
		opacity: 0.6;
	}
	.list {
		list-style: none;
		margin: 0;
		padding: 0;
		border-top: 1px solid rgba(242, 240, 234, 0.12);
	}
	li {
		border-bottom: 1px solid rgba(242, 240, 234, 0.12);
		transition: opacity 0.5s ease;
	}
	.hovering li:not(.active) {
		opacity: 0.32;
	}
	.row {
		display: grid;
		grid-template-columns: 6.4rem 1fr 20rem 5.6rem;
		gap: 2.4rem;
		align-items: center;
		padding: 1.4rem 0.8rem;
		color: inherit;
		text-decoration: none;
	}
	.row__i,
	.row__kind,
	.row__year {
		font-family: var(--font-mono);
		font-size: 1.2rem;
		letter-spacing: 0.04em;
		opacity: 0.5;
	}
	.row__year {
		text-align: right;
	}
	.row__title {
		font-family: var(--font-display);
		font-style: italic;
		font-size: clamp(5.6rem, 8vw, 11.2rem);
		line-height: 0.9;
		transition: transform 0.6s cubic-bezier(0.19, 1, 0.22, 1);
	}
	.active .row__title {
		transform: translateX(4.8rem);
	}
	.spacer {
		height: 30vh;
	}

	/* floating card that trails the cursor and flips per row */
	.card-anchor {
		position: fixed;
		top: 0;
		left: 0;
		width: 0;
		height: 0;
		pointer-events: none;
		z-index: 5;
		perspective: 1400px;
	}
	.card {
		position: absolute;
		left: 4rem;
		top: -17.5rem;
		width: 28rem;
		height: 35rem;
		transform: rotateZ(-5deg) rotateY(var(--ry)) scale(0.6);
		opacity: 0;
		transform-style: preserve-3d;
		transition:
			transform 1.1s cubic-bezier(0.19, 1, 0.22, 1),
			opacity 0.4s ease;
	}
	.show .card {
		opacity: 1;
		transform: rotateZ(-5deg) rotateY(var(--ry)) scale(1);
	}
	.card__face {
		position: absolute;
		inset: 0;
		transform: rotateY(var(--face));
		display: flex;
		align-items: flex-end;
		justify-content: space-between;
		padding: 1.2rem 1.4rem;
		font-family: var(--font-mono);
		font-size: 1.2rem;
		color: var(--text);
		background:
			radial-gradient(circle at 50% 45%, rgba(255, 255, 255, 0.22), transparent 60%),
			linear-gradient(var(--a), var(--b));
		opacity: 0;
		transition: opacity 0.35s ease;
	}
	.card__face.on {
		opacity: 1;
	}

	@media (max-width: 800px) {
		.row {
			/* minmax(0, …): long titles can't push the row wider than the screen */
			grid-template-columns: 3.2rem minmax(0, 1fr) 4.8rem;
			gap: 1.2rem;
		}
		.row__title {
			font-size: clamp(3.6rem, 12vw, 5.6rem);
			overflow-wrap: anywhere;
		}
		.row__kind {
			display: none;
		}
		.card-anchor {
			display: none;
		}
	}
	@media (prefers-reduced-motion: reduce) {
		.letter > span,
		.card {
			transition-duration: 0.01s;
		}
	}
</style>
