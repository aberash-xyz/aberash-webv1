<script>
	import { fade, fly } from 'svelte/transition';
	import { backOut, linear } from 'svelte/easing';
	import { flip } from 'svelte/animate';
	import { onMount } from 'svelte';

	let visible = false;
	let collapse = false;

	const letters = ['a', 'b', 'e', 'r', 'a', 's', 'h', '.'];
	onMount((_) => {
		const showInterval = setTimeout((_) => {
			visible = true;
		}, 500);

		const hideInterval = setTimeout((_) => {
			// hide everything but first and last letter
			collapse = true;
		}, 5000);

		return () => {
			clearInterval(showInterval);
			clearInterval(hideInterval);
		};
	});
</script>

<div>
	{#if visible}
		<h1>
			{#each letters as letter, i}
				{#if !collapse || i === 0 || i === letters.length - 1}
					<span
						in:fly={{
							y: 100,
							delay: i * 0,
							duration: 250,
							easing: linear
						}}
						out:fly={{
							delay: i * 100,
							x: 0,
							easing: backOut
						}}
						>{letter}
					</span>
				{/if}
			{/each}
		</h1>
	{/if}
</div>

<style>
	div {
		height: 100%;
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		overflow: hidden;
		background: var(--primary);
	}
	h1 {
		overflow: hidden;
		color: var(--text);
	}

	span {
		font-size: 9.2rem;
		font-weight: 500;
		display: inline-block;
		background: rgba(0, 0, 0, 0.008);
		margin: 0.1rem;
	}

	@media (max-width: 740px) {
		span {
			font-size: 5.2rem;
			margin: 0.1rem;
			font-weight: 500;
			/*			color: rgba(0,0,0,0.7);*/
		}
	}
</style>
