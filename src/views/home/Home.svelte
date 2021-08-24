<script>
	import { load } from 'opentype.js'
	import { fly } from 'svelte/transition'
	import { onMount } from 'svelte'
	import anime from 'animejs'

	let gloabalLoad = false

	let textSvg = {
		load: false,
		viewBox: null,
		path: null,
		fill: false
	}

	onMount(async () => {
		gloabalLoad = true
	});

	// Generates SVG from user input.
	async function make(value) {
		let input = value.target.querySelector('input').value
		if (!input || input.length < 2) return

    let font = await load('https://fonts.gstatic.com/s/roboto/v27/KFOmCnqEu92Fr1Mu4mxP.ttf') // Google font.
    let path = await font.getPath(`Welcome ${input}!`, 0, 150, 72)
		let viewBoxValues = Object.values(path.getBoundingBox())
		let adjustedViewBox = [viewBoxValues[0]-6, viewBoxValues[1]-6, viewBoxValues[2]+12, viewBoxValues[3]-85]

		textSvg.viewBox = adjustedViewBox
		textSvg.path = path.toPathData()
		textSvg.load = true
	}

	// Draw line animation.
	async function drawLine(e, { duration = 1500, delay = 0 }) {
		// Fix for Firefox.
		let maximumWait = 100
		while(e.getTotalLength() === 3.99999987306209e-20 && maximumWait > 0) {
			await new Promise(resolve => setTimeout(resolve, 1))
			maximumWait--
		}
		anime({
			targets: e,
			strokeDashoffset: [anime.setDashoffset, 0],
			easing: 'easeInOutSine',
			duration: duration,
			delay: delay,
			direction: 'normal',
			loop: false
		})
	}
</script>

<svelte:head> <!-- Page title -->
	<title>Home</title>
</svelte:head>

{#if gloabalLoad && !textSvg.load} <!-- User input form -->
	<form id="name-form" autocomplete="off"
		out:fly="{{ duration: 750, y: -300 }}"
		on:submit|preventDefault="{make}">
		<svg>
			<rect in:drawLine="{{ duration: 1000 }}"/>
		</svg>
		<label for="name-input"><p>Name:</p></label>
		<div id="input-holder">
			<input id="name-input" type="text" placeholder="Enter your name..."/>
			<button type="submit">Enter</button>
		</div>
	</form>
{/if}

{#if textSvg.load} <!-- Generated text SVG -->
	<div id="svg-holder" in:fly="{{ duration: 750, y: 300 }}">
		<svg viewbox="{textSvg.viewBox}">
			<path d="{textSvg.path}" on:introstart="{setTimeout(()=>textSvg.fill=true,750)}" in:drawLine="{{ duration: 3500 }}" class:fill-svg="{textSvg.fill}" />
		</svg>
	</div>
{/if}

<style> /** CSS styles **/
	#name-form {
		padding: 15.5px;
		display: table;
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
	}

	#name-form svg {
		position: absolute;
    top: 0px;
    left: 0;
    width: 100%;
    height: 100%;
		pointer-events: none;
	}

	#name-form svg rect {
		fill: none;
    stroke-width: 3px;
    stroke: #000;
    width: inherit;
    height: inherit;
	}

	#name-form label p {
		margin: 0;
    font-size: 17px;
    font-variant: all-small-caps;
	}

	#input-holder {
		display: flex;
    flex-direction: row;
    flex-wrap: nowrap;
	}

	#name-input {
		border: 1px dashed #000;
    border-radius: 0;
    outline: none;
    display: inline;
    background: #fff;
	}

	#name-form button {
		border: 1px dashed #000;
    position: relative;
    left: -1px;
    background: #fff;
		cursor: pointer;
	}

	#svg-holder {
		width: 500px;
		max-width: 100%;
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
	}

	#svg-holder svg path {
		fill: transparent;
		stroke: #000;
		stroke-width: 3px;
		transition: 1s fill;
		stroke-linecap: square;
	}

	.fill-svg {
		fill: #000!important;
	}
</style>