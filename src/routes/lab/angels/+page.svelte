<script>
	import { onMount, tick } from 'svelte';
	import LabHint from '$lib/LabHint.svelte';
	import LabNav from '$lib/LabNav.svelte';

	// feel: tweak these live
	const REACH = { x: 18, y: 10 }; // max iris travel, svg units
	const NEAR = 180; // cursor closer than this eases the iris back toward centre
	const EASE = 12; // higher = snappier
	const SPIN = 2; // flower turn per degree the cursor travels around it
	const SPIN_EASE = 5;
	const TWIST = 0.25; // helix slide per svg unit of cursor travel (top/bottom: x, sides: y)
	const TWIST_EASE = 5;
	const LEAN = 0.7; // how far a tooth tip leans toward the cursor, × its own width
	const LEAN_EASE = 6;
	const RIPPLE = 420; // svg units between checkerboard pulses; smaller = busier
	const RIPPLE_MIN = 0.4; // smallest a square shrinks to
	const RIPPLE_EASE = 6;
	// [cx, cy, scale] svg units. side tiles are shorter than their flowers, so those shrink to fit and don't get cut off as they turn
	const FLOWERS = [
		[70, 70, 1], [835.5, 70, 1], [1370, 70, 1],
		[70, 513.7, 0.76], [1370, 513.7, 0.76],
		[70, 830, 1], [835.5, 830, 1], [1370, 830, 1]
	];

	let host;
	let art = '';
	let cross; // follows the mouse in place of the cursor

	const ns = 'http://www.w3.org/2000/svg';

	// <g> clipped to a tile, just inside its outline
	const clipTo = (b, id) => {
		const clip = document.createElementNS(ns, 'clipPath');
		clip.id = id;
		const rect = document.createElementNS(ns, 'rect');
		const box = { x: b.x + 2, y: b.y + 2, width: b.width - 4, height: b.height - 4 };
		for (const k in box) rect.setAttribute(k, box[k]);
		clip.append(rect);
		const g = document.createElementNS(ns, 'g');
		g.setAttribute('clip-path', `url(#${id})`);
		g.append(clip);
		return g;
	};

	// green triangles in the border, three per tile, right after their tile
	const buildTeeth = (border) => {
		const els = border.filter((el) => el.getAttribute('fill') === '#2C6A4C' && el.hasAttribute('stroke'));
		const out = [];
		for (let i = 0; i < els.length; i += 3) {
			const trio = els.slice(i, i + 3);
			const tile = trio[0].previousElementSibling.getBBox();
			const vertical = trio[0].getBBox().width > trio[0].getBBox().height;
			const g = clipTo(tile, `teeth-${i}`);
			trio[0].before(g);
			g.append(...trio);
			for (const el of trio) {
				const b = el.getBBox();
				out.push({
					el, vertical, lean: 0,
					cx: b.x + b.width / 2,
					cy: b.y + b.height / 2,
					width: vertical ? b.height : b.width,
					height: vertical ? b.width : b.height,
					base: vertical ? b.x : b.y + b.height
				});
			}
		}
		return out;
	};

	// checkerboard squares: small, black, straight-edged, no outline
	const buildSquares = (border) =>
		border
			.filter((el) => el.getAttribute('fill') === '#14121C' && !el.hasAttribute('stroke') && !el.getAttribute('d').includes('C'))
			.map((el) => {
				const b = el.getBBox();
				return { el, scale: 1, cx: b.x + b.width / 2, cy: b.y + b.height / 2 };
			});

	// strands are the only thick-stroked paths: 4 per half turn (outline + colour, × 2 strands), right after their tile.
	// group each run, clip it to its tile and add one spare turn so it can slide forever
	const buildHelixes = (svg) => {
		const runs = [];
		let before = null;
		for (const el of svg.querySelectorAll('path')) {
			if (!el.closest('defs, mask') && +el.getAttribute('stroke-width') > 16) {
				if (runs.length && runs.at(-1).at(-1) === before) runs.at(-1).push(el);
				else runs.push([el]);
			}
			before = el;
		}
		return runs.map((strands, i) => {
			const b = strands[0].previousElementSibling.getBBox();
			const vertical = b.height > b.width;
			const axis = vertical ? 'y' : 'x';
			const period = 2 * (strands[4].getPointAtLength(0)[axis] - strands[0].getPointAtLength(0)[axis]);

			const outer = clipTo(b, `helix-${i}`);
			const g = document.createElementNS(ns, 'g');
			const spare = document.createElementNS(ns, 'g');
			spare.setAttribute('transform', vertical ? `translate(0 ${-period})` : `translate(${-period} 0)`);
			strands[0].before(outer);
			spare.append(...strands.slice(0, 8).map((el) => el.cloneNode()));
			g.append(spare, ...strands);
			outer.append(g);
			// opposite sides twist opposite ways
			return { g, vertical, period, dir: (vertical ? b.x > 720 : b.y > 450) ? -1 : 1, target: 0, pos: 0 };
		});
	};

	onMount(() => {
		let raf = 0;
		let dead = false;
		let last = 0;
		let pointer = null; // client coords; null = look straight ahead
		let svg;
		let eyes = [];
		let flowers = [];
		let helixes = [];
		let teeth = [];
		let squares = [];
		let prev = null; // last cursor position, svg units

		const onMove = (e) => {
			pointer = { x: e.clientX, y: e.clientY };
			// moved here, not in the loop, so it never lags the hand
			cross.style.transform = `translate(${e.clientX}px, ${e.clientY}px)`;
			cross.classList.toggle('show', e.pointerType === 'mouse');
		};
		const onLeave = () => {
			pointer = null;
			cross.classList.remove('show');
		};
		document.documentElement.classList.add('angels');

		const loop = (now) => {
			raf = requestAnimationFrame(loop);
			const k = 1 - Math.exp((-EASE * Math.min(now - last, 50)) / 1000);
			const kSpin = 1 - Math.exp((-SPIN_EASE * Math.min(now - last, 50)) / 1000);
			const kTwist = 1 - Math.exp((-TWIST_EASE * Math.min(now - last, 50)) / 1000);
			const kLean = 1 - Math.exp((-LEAN_EASE * Math.min(now - last, 50)) / 1000);
			const kRipple = 1 - Math.exp((-RIPPLE_EASE * Math.min(now - last, 50)) / 1000);
			last = now;
			const ctm = pointer && svg.getScreenCTM();
			const p = ctm && new DOMPoint(pointer.x, pointer.y).matrixTransform(ctm.inverse());
			for (const eye of eyes) {
				let tx = 0;
				let ty = 0;
				if (p) {
					const dx = p.x - eye.cx;
					const dy = p.y - eye.cy;
					const d = Math.max(Math.hypot(dx, dy), NEAR);
					tx = (dx / d) * REACH.x;
					ty = (dy / d) * REACH.y;
				}
				eye.x += (tx - eye.x) * k;
				eye.y += (ty - eye.y) * k;
				for (const part of eye.parts) part.setAttribute('transform', `translate(${eye.x} ${eye.y})`);
			}
			for (const f of flowers) {
				// like turning a dial: follow the change in cursor angle around the flower
				const angle = p ? (Math.atan2(p.y - f.cy, p.x - f.cx) * 180) / Math.PI : null;
				if (angle !== null && f.angle !== null) {
					const d = angle - f.angle;
					f.target += (d - 360 * Math.round(d / 360)) * SPIN; // shortest way round, no jump at ±180
				}
				f.angle = angle;
				f.rot += (f.target - f.rot) * kSpin;
				for (const part of f.parts) part.setAttribute('transform', `translate(${f.cx} ${f.cy}) rotate(${f.rot}) scale(${f.scale}) translate(${-f.cx} ${-f.cy})`);
			}
			for (const h of helixes) {
				if (p && prev) h.target += (h.vertical ? p.y - prev.y : p.x - prev.x) * TWIST * h.dir;
				h.pos += (h.target - h.pos) * kTwist;
				// a rotating helix seen side-on is its strands sliding along the axis; wrap every full turn
				const o = ((h.pos % h.period) + h.period) % h.period;
				h.g.setAttribute('transform', h.vertical ? `translate(0 ${o})` : `translate(${o} 0)`);
			}
			prev = p;
			for (const t of teeth) {
				let lean = 0;
				if (p) {
					const d = Math.max(Math.hypot(p.x - t.cx, p.y - t.cy), NEAR);
					lean = ((t.vertical ? p.y - t.cy : p.x - t.cx) / d) * LEAN * t.width;
				}
				t.lean += (lean - t.lean) * kLean;
				// shear about the base, so only the tip travels (side teeth point right, the rest point up)
				const k = t.lean / t.height;
				t.el.setAttribute('transform', t.vertical ? `matrix(1 ${k} 0 1 0 ${-k * t.base})` : `matrix(1 0 ${-k} 1 ${k * t.base} 0)`);
			}
			for (const q of squares) {
				let scale = 1;
				if (p) {
					const wave = Math.cos((Math.hypot(p.x - q.cx, p.y - q.cy) / RIPPLE) * Math.PI * 2);
					scale = RIPPLE_MIN + ((1 - RIPPLE_MIN) * (wave + 1)) / 2;
				}
				q.scale += (scale - q.scale) * kRipple;
				q.el.setAttribute('transform', `translate(${q.cx} ${q.cy}) scale(${q.scale}) translate(${-q.cx} ${-q.cy})`);
			}
		};

		const start = async () => {
			const res = await fetch('/lab/angels-idle.svg');
			if (!res.ok) throw new Error(`angels svg: ${res.status}`);
			const text = await res.text();
			if (dead) return;
			art = text;
			await tick();
			if (dead) return;
			svg = host.querySelector('svg');
			// figma export has no ids: masked group 0 is the panel, 1–6 are the eyes (iris + glint inside)
			eyes = [...svg.querySelectorAll('g[mask]')].slice(1).map((g) => {
				const b = g.firstElementChild.getBBox();
				return { parts: [...g.children], cx: b.x + b.width / 2, cy: b.y + b.height / 2, x: 0, y: 0 };
			});
			if (eyes.length !== 6) console.warn(`angels: expected 6 eyes, found ${eyes.length}`);
			// petals + centre: small paths whose middle falls inside a flower tile
			const paths = [...svg.querySelectorAll('path')].filter((el) => !el.closest('defs, mask')).map((el) => ({ el, b: el.getBBox() }));
			flowers = FLOWERS.map(([cx, cy, scale]) => ({
				cx, cy, scale, angle: null, target: 0, rot: 0,
				parts: paths
					.filter(({ b }) => b.width < 50 && b.height < 50 && Math.hypot(b.x + b.width / 2 - cx, b.y + b.height / 2 - cy) < 40)
					.map(({ el }) => el)
			}));
			if (flowers.some((f) => f.parts.length !== 9)) console.warn('angels: expected 9 parts per flower', flowers.map((f) => f.parts.length));
			const border = [...svg.querySelectorAll('path')].filter((el) => !el.closest('defs, mask, g[mask]'));
			teeth = buildTeeth(border);
			squares = buildSquares(border);
			if (teeth.length !== 12 || squares.length !== 64) console.warn(`angels: expected 12 teeth / 64 squares, found ${teeth.length} / ${squares.length}`);
			helixes = buildHelixes(svg);
			if (helixes.length !== 4) console.warn(`angels: expected 4 helixes, found ${helixes.length}`);
			raf = requestAnimationFrame(loop);
		};

		addEventListener('pointermove', onMove);
		addEventListener('pointerdown', onMove);
		document.documentElement.addEventListener('pointerleave', onLeave);
		start().catch(console.error);

		return () => {
			dead = true;
			document.documentElement.classList.remove('angels');
			cancelAnimationFrame(raf);
			removeEventListener('pointermove', onMove);
			removeEventListener('pointerdown', onMove);
			document.documentElement.removeEventListener('pointerleave', onLeave);
		};
	});
</script>

<svelte:head>
	<title>angels — lab — aberash.</title>
</svelte:head>

<div class="page">
	<div class="stage" bind:this={host} role="img" aria-label="three painted angels whose eyes follow the cursor">
		{@html art}
	</div>
	<LabHint follow={false} />
	<LabNav n={5} name="angels" />
</div>

<!-- ethiopian processional cross; its crossing point (50,50) sits on the pointer -->
<svg class="cross" bind:this={cross} viewBox="0 0 100 140" aria-hidden="true">
	{#each ['ink', 'gold'] as pass}
		<g class={pass}>
			<!-- staff, knop and the two brackets that carry the head -->
			<path class="solid" d="M47 86H53V138H47Z" />
			<path class="solid" d="M44 108H56L58 112L56 116H44L42 112Z" />
			<path class="line" d="M47 106C30 106 24 94 30 84C34 78 40 82 38 88" />
			<path class="line" d="M53 106C70 106 76 94 70 84C66 78 60 82 62 88" />
			<!-- openwork: diamond laced through a ring -->
			<path class="line" d="M50 17L83 50L50 83L17 50Z" />
			<circle class="line" cx="50" cy="50" r="22" />
			{#each [0, 90, 180, 270] as a}
				<g transform="rotate({a} 50 50)">
					<!-- flared arm, trefoil tip, loop on the diamond's shoulder -->
					<path class="solid" d="M50 50L46 44L43 22H57L54 44Z" />
					<path class="solid" d="M46 18L42 8H58L54 18Z" />
					<circle class="solid" cx="50" cy="5" r="3.6" />
					<circle class="solid" cx="41" cy="10" r="3" />
					<circle class="solid" cx="59" cy="10" r="3" />
					<circle class="line" cx="71.5" cy="28.5" r="5" />
				</g>
			{/each}
		</g>
	{/each}
	<path class="boss" d="M50 44L56 50L50 56L44 50Z" />
</svg>

<style>
	.page {
		--hint-y: calc(50% - var(--w) / 3.2); /* top edge of the panel, so the text never sits on the art */
		--w: min(58vw, calc((100dvh - 36rem) * 1.6));
		/* park (home) beside the panel instead of on top of it */
		--home-left: calc(50% + var(--w) / 2 + 1.6rem);
	}
	.stage {
		position: fixed;
		top: 50%;
		left: 50%;
		width: var(--w);
		aspect-ratio: 1440 / 900;
		transform: translate(-50%, -50%);
	}
	.stage :global(svg) {
		display: block;
		width: 100%;
		height: 100%;
	}
	.cross {
		position: fixed;
		top: 0;
		left: 0;
		width: 5rem;
		margin: -2.5rem 0 0 -2.5rem; /* 50/100 of the width, both axes */
		overflow: visible;
		pointer-events: none;
		opacity: 0;
		z-index: 20;
	}
	.cross:global(.show) {
		opacity: 1;
	}
	.ink .solid,
	.ink .line {
		fill: none;
		stroke: #14121c;
		stroke-width: 5;
		stroke-linejoin: round;
	}
	.ink .line {
		stroke-width: 8;
	}
	.gold .solid {
		fill: #e3bb4a;
	}
	.gold .line {
		fill: none;
		stroke: #e3bb4a;
		stroke-width: 3.4;
	}
	.boss {
		fill: #db4229;
		stroke: #14121c;
		stroke-width: 1.6;
	}
	/* the cross replaces the cursor and the site's trailing square, on this page only */
	@media (hover: hover) and (pointer: fine) {
		:global(html.angels),
		:global(html.angels *) {
			cursor: none !important;
		}
	}
	/* a finger drag steers the gaze instead of panning; pinch still zooms */
	:global(html.angels) {
		touch-action: pinch-zoom;
	}
	:global(html.angels .shadow) {
		display: none;
	}
	@media (max-width: 800px) {
		.page {
			--w: min(90vw, calc((100dvh - 36rem) * 1.6));
			--home-left: calc(50% - 5rem);
			--home-top: calc(50% + var(--w) / 3.2 + 4rem);
		}
	}
</style>
