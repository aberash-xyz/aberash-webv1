<script>
	import { onMount } from 'svelte';
	import Matter from 'matter-js';
	var Engine = Matter.Engine,
		Render = Matter.Render,
		Runner = Matter.Runner,
		Bodies = Matter.Bodies,
		Common = Matter.Common,
		Composite = Matter.Composite,
		Composites = Matter.Composites;

	// create an engine
	var engine = Engine.create();
	var world = engine.world;
	const letters = ['a', 'b', 'e', 'r', 'a', 's', 'h', '.'];
	onMount(async () => {
		const stage = document.getElementById('stage');
		const { x, y, width, height } = stage.getBoundingClientRect();
		console.log(x, y, width, height);
		// module aliases

		// create a renderer
		var render = Render.create({
			element: stage,
			engine: engine,
			options: {
				enabled: false,
				wireframes: false,
				background: '#f8e4de'
			}
		});
		engine.gravity.y = -1;

		// (x,y,w, h)
		var boxA = Bodies.rectangle(width / 2, 0, width, 50, { isStatic: true, background: '#dddddd' });
		var boxB = Bodies.rectangle(400, height - 50, 800, 50, { isStatic: true });
		var boxC = Bodies.rectangle(800, 0, 50, height * 2, { isStatic: true });
		var boxD = Bodies.rectangle(0, 300, 50, 600, { isStatic: true });

		// add all of the bodies to the world
		Composite.add(engine.world, [boxA, boxB, boxC, boxD]);

		var letterBox = Bodies.rectangle(400, 100, 50, 50);
		var letterBox2 = Bodies.rectangle(420, 150, 50, 50);
		// var stack = Composites.stack(50, 1, 5, 5, 0, 0, function (x, y) {
		// 	return Bodies.rectangle(x, y, Common.random(40, 100), Common.random(30, 80));
		// });

		Composite.add(world, [letterBox, letterBox2]);
		// run the renderer
		Render.run(render);

		// create runner
		var runner = Runner.create();

		// run the engine
		Runner.run(runner, engine);
	});

	function addRect() {
		var newBox = Bodies.rectangle(420 + Math.random() * 100, 300, 50, 50);
		Composite.add(world, newBox);
	}
</script>

<div id="stage" class="my-stage" />
<button on:click={addRect}> ADD RECT!</button>
<div class="letters-container">
	{#each letters as letter, i}
		<span class="letter" style={`left:${i * 150}px`}>{letter} </span>
	{/each}
</div>

<style>
	.my-stage {
		height: 100%;
		width: 100%;
		background: var(--primary);
	}

	button {
		position: absolute;
		bottom: 10%;
	}
	.letter {
		top: 20%;
		z-index: 2;
		font-size: 40rem;
		position: absolute;
		color: var(--secondary);
	}
</style>
