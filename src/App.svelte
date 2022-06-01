
<script>
	import Modal from "./Modal.svelte"
	import {newLocalStore} from "./store"
	import {onMount} from "svelte"
	const defaultOptions = {
		size: 5,
		screenX: 200,
		screenY: 200,
		move: true,
		moveSizeX: 50,
		moveSizeY: 50,
		moveSpeed: 600,
		movePause: 100,
		crosshair: 1,
	}
	let openProfile = false
	let loadProfileModal = false
	let config = newLocalStore("config", defaultOptions)
	let profileToLoad = newLocalStore("currentProfile", "")
	let currentProfileName = $profileToLoad
	let profiles = newLocalStore("profiles", {})
	let moveX = 0
	let moveY = 0
	let svg
	let start, end
	let lastResult = null
	let dot
	let misses = 0
	let missesResults = []
	let lastMisses = null
	let results = []
	let once = false
	let shown = true
	let screenW, screenH
	let cooldown = false
	let moveInterval
	let moveTimeout
	let saveWarning = false
	let deleteWarning = false
	
	function setDefaults(){
		$profileToLoad = ""
		currentProfileName = ""
		$config = defaultOptions
		setTimeout(restart, 50)
	}
	 
	function getAverage(values, round = true){
		let sum = values.reduce((acc, val) => acc+val, 0)
		if(round){
			return Math.round(values.length < 1 ? 0 : sum / values.length)
		}
		return values.length < 1 ? 0 : sum / values.length
	}
	function handleMisses(e){
		if(cooldown || !shown)return
		if(e.target.id != "dot" && once){
			misses += 1
		}
	}
	function handleClick(e){
		if(cooldown || !shown)return
		cooldown = true
		setTimeout(() => cooldown = false, 200)
		let {width, height} = svg.getBoundingClientRect()
		hideAndCalculate(e.target)
		setTimeout(() => {
			newPosition(width, height, e.target)	
		}, random(400, 700))
		
	}
	
	function newPosition(width, height, node){
		show(node)
		let x = random(10, width - $config.size - $config.moveSizeX)
		let y = random(10, height - $config.size - $config.moveSizeY)
		node.setAttribute("cx", x)
		node.setAttribute("cy", y)
	}
	
	function hideAndCalculate(node){
		shown = false
		end = Date.now()
		node.style.opacity = "0"
		if(!once){
			once = true
		}else{
			missesResults = [...missesResults, misses]
			lastResult = end - start
			lastMisses = misses
			misses = 0
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
		lastMisses = null
		once = false
		misses = 0
		moveX = 0
		moveY = 0
		results = []
		missesResults = []
		init()
	}
	

	function init(){
		cleanup()
		moveInterval = setInterval(moveAnimation, $config.moveSpeed + $config.movePause)
		let {width, height} = svg.getBoundingClientRect()
		dot.setAttribute("cx", (width/2) - ($config.size/2))
		dot.setAttribute("cy", (height/2) - ($config.size/2))
	}
	function cleanup(){
		clearInterval(moveInterval)
	}

	function moveAnimation(){
		if(!$config.move || (!once)){
			moveX = 0
			moveY = 0
			return
		}
		moveX = random(0, $config.moveSizeX)
		moveY = random(0, $config.moveSizeY)
	}
	function updateInterval(speed, pause){
		clearTimeout(moveTimeout)
		moveTimeout = setTimeout(() => {
			console.log("updating")
			clearInterval(moveInterval)
			moveInterval = setInterval(moveAnimation, speed + pause)
		}, 750)
	}

	function openLoadProfile(){
		loadProfileModal = true
	}
	function loadProfile(){
		if($profiles[$profileToLoad]){
			$config = JSON.parse(JSON.stringify($profiles[$profileToLoad]))
			loadProfileModal = false
			currentProfileName = $profileToLoad
			setTimeout(restart, 50)
		}
	}

	function saveProfileModal(){
		openProfile = true
	}
	function confirmSaveProfile(){
		$profiles[currentProfileName] = JSON.parse(JSON.stringify($config))
		openProfile = false
		saveWarning = false
	}
	function cancelSave(){
		saveWarning = false
	}
	function deleteProfile(){
		deleteWarning = true
	}
	function saveProfile(){
		if(!currentProfileName)return
		if($profiles[currentProfileName]){
			saveWarning = true
		}else{
			$profiles[currentProfileName] = $config
			openProfile = false
		}
	}
	function confirmDeleteProfile(){
		if(!$profileToLoad)return
		delete $profiles[$profileToLoad]
		$profiles = $profiles
		deleteWarning = false
		loadProfileModal = false
		$profileToLoad = ""
	}
	function cancelDelete(){
		deleteWarning = false
	}

	$:{
		updateInterval($config.moveSpeed, $config.movePause)
	}
	

	onMount(() => {
		init()
		return cleanup
	})
	
</script>

<svelte:window on:keydown={handleKey} bind:innerWidth={screenW} bind:innerHeight={screenH}/>

<style>
	* {
		user-select: none;
	}
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
		position:relative;
		user-select:none;
		color: rgb(0,0,0,0.4);
	}
	.result{
		animation: out 700ms 1 forwards;
		position:absolute;
	}
	.grid{
		display:grid;
		grid-template-columns: 1fr 1fr;
		grid-gap:8px;
	}
	.crosshair > * {
		margin-right: 16px;
		padding: 0 8px;
		cursor:pointer;
		text-align:center;
	}
	button, select{
		width:100%;
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
	<main style="width: {$config.screenX}px;height:{$config.screenY}px; cursor: url('/crosshair-{[1,2,3,4].includes($config.crosshair) ? $config.crosshair : 1}.png') 12 12, auto;" on:mousedown={handleMisses}>
		{#key lastResult}
			<h1 class="result" style="color:rgb(255,0,0,0.4)">
				{lastResult || ""}{lastMisses !== null ? ` - ${lastMisses}` : ""}
			</h1>
		{/key}	
		<svg bind:this={svg}>
			<circle id="dot" style="transition:transform {$config.moveSpeed}ms ease-in-out;transform: translate({moveX}px, {moveY}px);"  bind:this={dot} on:mousedown={handleClick} r={$config.size} fill="red"/>
		</svg>
	</main>
	<div class="content">
		<h3>Profile: <span style="color:rgb(255,0,0,0.4)">{currentProfileName || "-"}</span></h3>
		<h3>
			Avg Speed: <span style="color:rgb(255,0,0,0.4)">{lastResult ? getAverage(results) : "Click red dot to start"} </span><br/>
		</h3>
		<h3>
			Missed clicks: <span style="color:rgb(255,0,0,0.4)">{lastResult ? getAverage(missesResults, false).toFixed(2) : ""} </span><br/>
		</h3>
		<div style="font-size:14px">
			<p>
				- Click red dot to start
			</p>
			<p>
				- Press Esc to reset
			</p>
			<p>
				- Left/Right to resize dot
			</p>
		</div>
		<div class="grid">
			<label>Dot Size
				<input type="number" bind:value={$config.size}/>
			</label>	
			<label>Horizontal Size
				<input type="number" bind:value={$config.screenX}/>
			</label>
			<label>Vertical Size
				<input type="number" bind:value={$config.screenY}/>
			</label>
			<p style="grid-column: 1 / -1; margin: 0;">Crosshair</p>
			<div class="crosshair" style="grid-column: 1 / -1; display: flex;">
				<label>1
					<input type="radio" bind:group={$config.crosshair} value={1}/>
				</label>
				<label>2
					<input type="radio" bind:group={$config.crosshair} value={2}/>
				</label>
				<label>3
					<input type="radio" bind:group={$config.crosshair} value={3}/>
				</label>
				<label>4
					<input type="radio" bind:group={$config.crosshair} value={4}/>
				</label>
			</div>	
			<label style="grid-column: 1 / -1; cursor:pointer;">Move
				<input type="checkbox" bind:checked={$config.move}/>
			</label>
			{#if $config.move}
				<label>Horizontal Move Distance
					<input type="number" bind:value={$config.moveSizeX}/>
				</label>	
				<label>Vertical Move Distance
					<input type="number" bind:value={$config.moveSizeY}/>
				</label>	
				<label>Move Speed
					<input type="number" bind:value={$config.moveSpeed}/>
				</label>	
				<label>Move Pause
					<input type="number" bind:value={$config.movePause}/>
				</label>	
			{/if}

			<button style="grid-column: 1 / -1" on:click={openLoadProfile}>
				Load Profile
			</button>
			<button style="grid-column: 1 / -1" on:click={saveProfileModal}>
				Save Profile
			</button>
			<button style="grid-column: 1 / -1" on:click={setDefaults}>
				Set to default
			</button>
		</div>
	</div>

</div>


<Modal bind:open={openProfile} closeOnBackdrop style="padding:32px; background-color:white;">	
	{#if saveWarning}
		<p>The profile <strong>{currentProfileName}</strong> already exists, do you want to overwrite?</p>
		<button on:click={confirmSaveProfile}>
			Confirm
		</button>	
		<button on:click={cancelSave}>
			Cancel
		</button>	
	{:else}
		<label>Profile Name
			<input bind:value={currentProfileName}/>
		</label>
		<button on:click={saveProfile}>
			Save
		</button>	
	{/if}
</Modal>

<Modal bind:open={loadProfileModal} closeOnBackdrop style="padding:32px; background-color:white;">	
	{#if deleteWarning}
		<p>Delete <strong>{$profileToLoad}</strong>?</p>
		<button on:click={confirmDeleteProfile}>
			Confirm
		</button>	
		<button on:click={cancelDelete}>
			Cancel
		</button>	
	{:else}
		<label>
			Profiles<br/>
			<select bind:value={$profileToLoad}>
				{#each Object.keys($profiles) as profile}
					<option value={profile}>
						{profile}
					</option>
				{/each}
			</select>
		</label>
		<button on:click={loadProfile}>
			Load
		</button>	
		<button on:click={deleteProfile}>
			Delete
		</button>	
	{/if}
</Modal>