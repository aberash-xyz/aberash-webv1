<script>
	const images = [
		'https://i.pinimg.com/1200x/e6/47/31/e64731ccabe38cdfe06b6493c5fee2a3.jpg',
		'https://i.pinimg.com/736x/94/44/0d/94440d0391170c194f2c261ed529828d.jpg',
		'https://i.pinimg.com/736x/ef/5c/92/ef5c92580d27c3ef7b63f410928d52b0.jpg',
		'https://i.pinimg.com/1200x/8f/4b/53/8f4b53fc612b211bf5660029e9208190.jpg',
		'https://i.pinimg.com/736x/f5/ac/7a/f5ac7ada03d1b90213808aa0cb751d2b.jpg',
		'https://i.pinimg.com/736x/37/ff/d4/37ffd48e6a6e27f7082bfcc27083acc5.jpg',
		'https://i.pinimg.com/736x/7a/4f/5d/7a4f5dfcdd626f0d6940a62fddd5d12f.jpg',
		'https://i.pinimg.com/1200x/69/4b/72/694b728fef709530961d56646692d6e1.jpg',
    	'https://i.pinimg.com/control1/736x/e6/58/45/e658459c09faff1e7bcf17d734d5cfae.jpg',
    	'https://i.pinimg.com/736x/13/1d/12/131d12bdf13b7f6b10415d56fe786e67.jpg',
    	'https://i.pinimg.com/control1/736x/0d/fc/da/0dfcdaafabbfe78a8d08713c379a0921.jpg',
    	'https://i.pinimg.com/control1/736x/87/1c/58/871c58276410ec463ad25a7d6c817563.jpg'
	];
</script>

<div class="container">
	<h1 class="title"><a class="title__link" href="/"><span class="hover-block" /> (home)</a></h1>
	<div class="grid">
		{#each images as image}
			<img class="image" src={image} alt="Mood" />
		{/each}
	</div>
</div>

<style>
	.container {
		padding-top: calc(2 * var(--margin));
		box-sizing: border-box;
	}
	.grid {
		padding: 1.2rem 1.6rem;
		display: grid;
		grid-template-columns: repeat(4, minmax(200px, 1fr));
		grid-auto-columns: 1fr;
		grid-column-gap: 8px;
		pointer-events: none;
		z-index: 0;
	}
	.image {
		aspect-ratio: 1/1;
		height: 300px;
		width: 300px;
		object-fit: contain;
		pointer-events: none;
		z-index: 0;
	}
	.title {
		font-size: 4.8rem;
		color: var(--text);
		text-decoration: none;
		position: absolute;
		top: 45%;
		left: 50%;
		transform: transform(-50%, -50%);
		cursor: pointer;
		z-index: 1;
	}
	.title__link {
		font-family: var(--font-mono);
		color: inherit;
		text-decoration: none;
		cursor: pointer;
		position: relative;
		overflow: hidden;
		display: block;
	}
	.hover-block {
		height: 100%;
		width: 100%;
		background: var(--hover-color);
		position: absolute;
		left: -100%;
		transition: ease-in-out 0.3s;
	}
	.title__link:hover .hover-block {
		left: 100%;
	}
	@media (max-width: 800px) {
		.grid {
			grid-template-columns: repeat(1, 1fr);
		}
		.title {
			top: 50%;
			position: fixed;
		}
		.image {
			width: 100%;
			height: auto;
		}
	}
</style>
