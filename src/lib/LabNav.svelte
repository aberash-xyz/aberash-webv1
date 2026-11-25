<script>
	// shared chrome for every /lab/* page: "lab / 0n — name" (lab → grid) + (home)
	export let n;
	export let name;
</script>

<p class="label"><a href="/lab">lab</a> / {String(n).padStart(2, '0')} — {name}</p>
<a class="home" href="/"><span class="hover-block"></span>(home)</a>

<style>
	.label {
		position: fixed;
		left: 50%;
		bottom: 13rem;
		transform: translateX(-50%);
		margin: 0;
		font-family: var(--font-mono);
		font-size: 1.3rem;
		color: rgba(242, 240, 234, 0.5);
		z-index: 1;
	}
	.label a {
		color: var(--text);
		text-decoration: underline;
		text-underline-offset: 0.3em;
		text-decoration-color: rgba(242, 240, 234, 0.4);
		transition: text-decoration-color 0.3s ease;
	}
	.label a:hover {
		text-decoration-color: var(--hover-color);
	}
	/* same (home) link as /mood: violet block wipes across on hover */
	.home {
		position: fixed;
		top: var(--home-top, 50%);
		left: var(--home-left, 62%);
		transform: translateY(-50%);
		overflow: hidden;
		padding: 0.4rem 0.8rem;
		font-family: var(--font-mono);
		font-size: 2.4rem;
		color: var(--text);
		text-decoration: none;
		z-index: 1;
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
</style>
