<script>
	import { onMount } from 'svelte';
	import Matter from 'matter-js';

	let canvas;
	let engine;
	let render;
	let world;
	let walls = [];

	onMount(() => {
		// Create engine with inverted gravity and general horizontal drift
		engine = Matter.Engine.create({
			gravity: {
				x: 0, // General drift force to the right
				y: -1, // Negative gravity makes things float up
				scale: 0.001
			}
		});

		world = engine.world;

		// Create renderer
		render = Matter.Render.create({
			element: canvas,
			engine: engine,
			options: {
				width: window.innerWidth,
				height: window.innerHeight,
				wireframes: false,
				background: 'transparent'
			}
		});

		// Create walls
		const createWalls = () => {
			const wallThickness = 50;
			return [
				// Left wall
				Matter.Bodies.rectangle(
					-wallThickness / 2,
					window.innerHeight / 2,
					wallThickness,
					window.innerHeight,
					{
						isStatic: true,
						render: { visible: false }
					}
				),
				// Right wall
				Matter.Bodies.rectangle(
					window.innerWidth + wallThickness / 2,
					window.innerHeight / 2,
					wallThickness,
					window.innerHeight,
					{
						isStatic: true,
						render: { visible: false }
					}
				),
				// Top boundary (to remove boxes that escape)
				Matter.Bodies.rectangle(
					window.innerWidth / 2,
					-wallThickness / 2,
					window.innerWidth,
					wallThickness,
					{
						isStatic: true,
						render: { visible: false }
					}
				)
			];
		};

		// Add invisible boundaries at edges
		walls = createWalls();
		Matter.World.add(world, walls);

		// Run the engine
		Matter.Engine.run(engine);
		Matter.Render.run(render);

		// Handle window resize
		const handleResize = () => {
			render.canvas.width = window.innerWidth;
			render.canvas.height = window.innerHeight;
			render.options.width = window.innerWidth;
			render.options.height = window.innerHeight;

			// Remove old walls
			Matter.World.remove(world, walls);

			// Create and add new walls at updated positions
			walls = createWalls();
			Matter.World.add(world, walls);
		};

		window.addEventListener('resize', handleResize);

		// Cleanup on destroy
		return () => {
			Matter.Render.stop(render);
			Matter.Engine.clear(engine);
			window.removeEventListener('resize', handleResize);
		};
	});

	const handleClick = (event) => {
		if (!engine) return;

		const x = event.clientX;
		const y = event.clientY;

		// Create a new box at click position
		const size = Math.random() * 80 + 20; // Random size between 20-50px
		const box = Matter.Bodies.rectangle(x, y, size, size, {
			restitution: 0.5,
			friction: 0.1,
			density: 0.002,
			frictionAir: 0.04,
			render: {
				fillStyle:
					getComputedStyle(document.documentElement).getPropertyValue('--text').trim() || '#000000'
			}
		});

		// Add initial upward force
		Matter.Body.applyForce(box, box.position, { x: 0, y: -0.00005 });

		// Add the box to the world
		Matter.World.add(world, box);

		// Remove boxes that go too far up
		setTimeout(() => {
			if (box.position.y < -100) {
				Matter.World.remove(world, box);
			}
		}, 10000);
	};
</script>

<div class="floating-boxes-container" on:click={handleClick}>
	<div bind:this={canvas} class="canvas-wrapper"></div>
</div>

<style>
	.floating-boxes-container {
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		pointer-events: all;
		z-index: 1;
	}

	.canvas-wrapper {
		width: 100%;
		height: 100%;
	}

	.canvas-wrapper :global(canvas) {
		position: fixed;
		top: 0;
		left: 0;
	}
</style>
