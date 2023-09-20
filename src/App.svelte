<script>
  import { onMount } from "svelte";


	let main, plane, milk, cat;
	
	let disabled = false;
	let right  = 0;
	let left = 0;
	let top = 0;

	$: planeStyle = `top: ${top}px; left: ${left}px; right: ${right}px`

	const sleep = (ms) => new Promise(r => setTimeout(r, ms));

	async function dropMilk() {
		disabled = true; 
		await sleep(200);
		disabled = false;
	}

	function changeHeight() {
		console.log('height');
	}

	function moveCat(){
		console.log('moveCat');
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
			// console.log(main.clientWidth, plane);
			if (pos >= (main.clientWidth - plane.clientWidth)) {
				lapCount = lapCount + 1;
				pos = 0;
				changeHeight();
				if (!(lapCount%5)){
					moveCat();
				}
			}
			// console.log(plane);
			left = pos;
		}

	}

	onMount(movePlane);
</script>

<main bind:this={main}>
	<button {disabled} on:click={dropMilk} class='move-button'>Drop Milk</button>
	<img src='plane.png' class='plane' alt='plane' style={planeStyle} bind:this={plane}/>
	<img src='milk.png' class='milk' alt='milk' bind:this={milk}/>
	<img src='pop-cat.gif' class='cat' alt='pop-cat' bind:this={cat}/>
</main>

<style>
	main {
		background-color: lightblue;
	}

	.plane {
		width: 69px;
		position: absolute;
	}

	.cat {
		width: 69px;
		position: absolute;
	}

	.milk {
		width: 42px;
		position: absolute;
	}
	
    .move-button {
    	position: bottom;
    }
</style>