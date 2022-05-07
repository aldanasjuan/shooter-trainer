
<script>
	import {onMount} from "svelte"
	let screenX = 500
	let screenY = 500
	let size = 10
	let svg
	let start, end
	let lastResult = null
	let dot
	let results = []
	let once = false
	let shown = true
	let screenW, screenH
	let cooldown = false
	 
	function getAverage(values){
		let sum = values.reduce((acc, val) => acc+val, 0)
		return Math.round(values.length < 1 ? 0 : sum / values.length)
	}
	
	function handleClick(e){
		if(cooldown || !shown)return
		cooldown = true
		setTimeout(() => cooldown = false, 200)
		let {width, height} = svg.getBoundingClientRect()
		hide(e.target)
		setTimeout(() => {
			newPosition(width, height, e.target)	
		}, random(400, 700))
		
	}
	
	function newPosition(width, height, node){
		show(node)
		let x = random(10, width - size)
		let y = random(10, height - size)
		node.setAttribute("cx", x)
		node.setAttribute("cy", y)
	}
	
	function hide(node){
		shown = false
		end = Date.now()
		node.style.opacity = "0"
		if(!once){
			once = true
		}else{
			lastResult = end - start
			results = [...results, lastResult]
		}
	}
	function show(node){
		shown = true
		start = Date.now()
		node.style.opacity = "1"
	}
	export function random(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min + 1) + min); 
}
	function handleKey(e){
		if(e.code == "Escape"){
			restart()
		}
		if(e.code == "ArrowLeft"){
			size -= 1
		}else if(e.code == "ArrowRight"){
			size += 1
		}
	}
	
	function restart(){
		lastResult = null
		once = false
		results = []
		init()
	}

	function init(){
		let {width, height} = svg.getBoundingClientRect()
		dot.setAttribute("cx", (width/2) - (size/2))
		dot.setAttribute("cy", (height/2) - (size/2))
	}

	onMount(() => {
		init()
	})
	
</script>

<svelte:window on:keydown={handleKey} bind:innerWidth={screenW} bind:innerHeight={screenH}/>

<style>
	:global(body, html), .container, svg{
		width: 100%;
		height: 100%;
	}
	.container{
		padding: 16px;
		display:grid;
		grid-auto-rows:min-content;
		grid-gap: 10px 0;
		place-items:center;
	}
	svg{
		position:relative;
	}
	label, input{
		display:block;
	}
	main{
		
		background-color:#ccc;
		display: grid;
		place-items:center;
		position:relative;
	}
	h1{
		line-height:1;
		margin: 10px 0;
		text-align:center;
		position:absolute;
		user-select:none;
		color: rgb(0,0,0,0.4);
	}
	.result{
		animation: out 700ms 1 forwards;
	}
	
	@keyframes out{
		from{
			opacity:1;
		}
		to{
			opacity: 0;
		}
	}
</style>

<div class="container">
	<main style="width: {screenX}px;height:{screenY}px">
	<h1>
		{#if lastResult}
			Avg Speed: <span style="color:rgb(255,0,0,0.4)">{getAverage(results)} </span><br/>
		{:else}
			Click red dot to start<br/>
		{/if}
		<span style="font-size:14px">
			Screen Size: {screenX} x {screenY}<br/> 
			Dot Size: {size} <br/>
			- Press Esc to reset<br/>
			- Left/Right to resize dot
		</span>
	</h1>
	{#key lastResult}
		<h1 class="result" style="top: 20%; color:rgb(255,0,0,0.4)">
			{lastResult || ""}
		</h1>
	{/key}
	<svg bind:this={svg}>
		<circle bind:this={dot} on:click={handleClick} r={size} fill="red"/>
	</svg>
</main>
	<label>Dot Size
		<input type="number" bind:value={size}/>
	</label>	
	<label>Horizontal Size
		<input type="number" bind:value={screenX}/>
	</label>
	<label>Vertical Size
		<input type="number" bind:value={screenY}/>
	</label>		
</div>
