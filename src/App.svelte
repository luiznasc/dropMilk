<script>
  	import { onMount } from "svelte";
	import { Mutex } from 'async-mutex';

	let mutex = new Mutex();

	let main, plane, milk, cat, score, milkCore;

	let display = 'none';
	let milkWidth = 42
	let catWidth = 69;

	let planeRight, planeLeft, planeTop, catLeft, catRight, milkLeft, milkTop;
	planeRight = planeLeft = planeTop = catLeft = catRight = milkLeft = milkTop = 0;

	$: planeStyle = `top: ${planeTop}px; left: ${planeLeft}px; right: ${planeRight}px`
	$: catStyle = `left: ${catLeft}px; right: ${catRight}px; width: ${catWidth}px;`
	$: milkStyle = `top: ${milkTop}px; left: ${milkLeft}px; display: ${display}; width: ${milkWidth}px;`

	const sleep = (ms) => new Promise(r => setTimeout(r, ms));

	async function dropMilk() {
		milkLeft = planeLeft;
		display = "block";
		for (let i = planeTop; i < main.clientHeight - (milk.clientHeight); i = i+2){
			await sleep();
			milkTop = i;
		}
		runScore();
		display = "none";
	}

	async function dropMilkQueue() {
		if (mutex.isLocked()) {
			return;
		}
		const release = await mutex.acquire();
		try {
			await dropMilk();
		} finally {
			release();
		}
	}


	function changeHeight() {
		planeTop= Math.floor(Math.random() * (main.clientHeight/2)); //Only half to make it more consistent
	}

	function moveCat(){
		// Some weird math for cat's position to prevent it from going too close
		// to the walls
		let catBox = Math.floor(Math.random() * (main.clientWidth - 2.5*cat.clientWidth));
		if (catBox < cat.clientWidth*2){
			catBox = cat.clientWidth*2;
		} else if (catBox > (main.clientWidth - (cat.clientWidth*2))) {
			catBox = main.clientWidth - (cat.clientWidth*2);
		}
		catLeft = catBox;
	}

	async function movePlane() {
		// for (var i = 0; i < (body.clientWidth - plane.clientWidth); i++) {
		//     await sleep(2);
		//     plane.style.left = i;
		// }
		let pos = 0;
		let lapCount = 0;
		while (true) {
			await sleep(2);
			pos = pos + 2;
			if (pos >= (main.clientWidth - plane.clientWidth)) {
				lapCount = lapCount + 1;
				pos = 0;
				changeHeight();
				if (!(lapCount%5)){
					moveCat();
				}
			}
			planeLeft = pos;
		}
	}

	function runScore() {
		milkCore = (milkLeft +milkWidth + milkLeft)/2;
		if ((milkCore > catLeft) && (milkCore <= catLeft+catWidth)){
			score.innerHTML++;
			moveCat();
		} else {
			score.innerHTML = 0;
		}
	}

	onMount(movePlane);
	onMount(moveCat);
</script>

<main bind:this={main} on:click={dropMilkQueue} on:keydown={dropMilkQueue}>
	<img src='plane.png' class='plane' alt='plane' style={planeStyle} bind:this={plane}/>
	<img src='milk.png' class='milk' alt='milk' {display} style={milkStyle} bind:this={milk}/>
	<img src='pop-cat.gif' class='cat' alt='pop-cat' style={catStyle} bind:this={cat}/>
	<div>Score: <div bind:this={score}>0</div></div>
</main>

<style>
	main {
		height: 100%;
		width: 100%;
		background-color: lightblue;
		text-align: center;
	}

	.plane {
		width: 69px;
		position: absolute;
	}

	.cat {
		position: absolute;
		bottom: 0px;
	}

	.milk {
		position: absolute;
		display: none;
	}
</style>