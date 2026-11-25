<script>
	// "(move your mouse)" / "(drag)": shown on load and whenever the pointer has been still for a while
	import { onMount } from 'svelte';

	export let follow = true; // sit at the pointer; false = stay at --hint-x / --hint-y
	export let idle = 5; // seconds

	let hint = { x: 0, y: 0, show: false, touch: false };

	onMount(() => {
		hint = { show: true, x: innerWidth / 2, y: innerHeight / 2, touch: matchMedia('(pointer: coarse)').matches };

		let timer;
		const onMove = (e) => {
			hint = { x: e.clientX, y: e.clientY, show: false, touch: e.pointerType !== 'mouse' };
			clearTimeout(timer);
			timer = setTimeout(() => (hint = { ...hint, show: true }), idle * 1000);
		};
		addEventListener('pointermove', onMove);
		addEventListener('pointerdown', onMove);

		return () => {
			clearTimeout(timer);
			removeEventListener('pointermove', onMove);
			removeEventListener('pointerdown', onMove);
		};
	});
</script>

<p class="hint" class:show={hint.show} style={follow ? `left:${hint.x}px; top:${hint.y}px` : ''}>
	{hint.touch ? '(drag)' : '(move your mouse)'}
</p>

<style>
	.hint {
		position: fixed;
		left: var(--hint-x, 50%);
		top: var(--hint-y, 50%);
		margin: 0;
		transform: translate(-50%, calc(-100% - 1.6rem));
		font-family: var(--font-mono);
		font-size: 1.2rem;
		color: rgba(242, 240, 234, 0.6);
		white-space: nowrap;
		pointer-events: none;
		opacity: 0;
		transition: opacity 0.25s ease;
	}
	.hint.show {
		opacity: 1;
		transition-duration: 1.2s;
	}
</style>
