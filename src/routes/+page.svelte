<script lang="ts">
	import { onMount } from 'svelte';
	import { fade } from 'svelte/transition';

	let playerX = 100;
	let playerY = 100;
	let lastTime = 0;
	let area = 0;
	let playerSpeed = 5;
	let ready = false;
	let npcs = [];
	let nearbyNPC = null;
	let showNPCMessage = false;

	const addEnemies = () => {
		npcs.push({
			x: 200,
			y: 300,
			area: 0,
			message: 'Hello, I am an NPC!',
			background: 'blue'
		});
		npcs.push({
			x: 100,
			y: 100,
			area: 1,
			message: 'Hello, I am an NPC in area 1!',
			background: 'blue'
		});
		npcs.push({
			x: 200,
			y: 100,
			area: 1,
			message: 'Hello, I am an NPC in area 1!',
			background: 'blue'
		});
		npcs.push({
			x: 300,
			y: 100,
			area: 3,
			message: 'Hello, I am an NPC in area 1!',
			background: 'blue'
		});
	};

	const addWalls = () => {
		//add brown walls to left side of area 1
		for (let i = 0; i < 500; i += 32) {
			npcs.push({
				x: 0,
				y: i,
				area: 0,
				background: 'brown'
			});
		}
		//add brown walls to top of all areas
		for (let i = 0; i < 500; i += 32) {
			npcs.push({
				x: i,
				y: 0,
				area: 0,
				background: 'brown'
			});
			npcs.push({
				x: i,
				y: 0,
				area: 1,
				background: 'brown'
			});
			npcs.push({
				x: i,
				y: 0,
				area: 2,
				background: 'brown'
			});
			npcs.push({
				x: i,
				y: 0,
				area: 3,
				background: 'brown'
			});
		}
		//add brown walls to right side of area 1
	};

	const move = () => {
		if (keys.w) {
			playerY -= playerSpeed;
		}
		if (keys.a) {
			playerX -= playerSpeed;
		}
		if (keys.s) {
			playerY += playerSpeed;
		}
		if (keys.d) {
			playerX += playerSpeed;
		}
		sideScroll();
	};

	const keys = {
		w: false,
		a: false,
		s: false,
		d: false
	};

	const handleKeyDown = (event) => {
		if (event.key === 'w') {
			keys.w = true;
		}
		if (event.key === 'a') {
			keys.a = true;
		}
		if (event.key === 's') {
			keys.s = true;
		}
		if (event.key === 'd') {
			keys.d = true;
		}
	};

	const handleKeyUp = (event) => {
		if (event.key === 'w') {
			keys.w = false;
		}
		if (event.key === 'a') {
			keys.a = false;
		}
		if (event.key === 's') {
			keys.s = false;
		}
		if (event.key === 'd') {
			keys.d = false;
		}
	};

	onMount(() => {
		const gameLoop = () => {
			move();
			detectNearbyNPC();
			requestAnimationFrame(gameLoop);
			detectCollisions();
		};

		addEnemies();
		addWalls();
		gameLoop();

		ready = true;
	});

	//if player goes offscreen, side scroll
	const sideScroll = () => {
		if (playerX > 500 - 32) {
			area += 1;
			playerX = 0;
		}
		if (playerX < 0 && area > 0) {
			area -= 1;
			playerX = 500 - 32;
		}
		if (playerX < 0 && area === 0) {
			playerX = 0;
		}
		//do not allow player to go offscreen vertically
		if (playerY > 500 - 32) {
			playerY = 500 - 32;
		}
		if (playerY < 0) {
			playerY = 0;
		}
	};

	const detectNearbyNPC = () => {
		//detect if player is within 100px of an npc
		nearbyNPC =
			npcs.find((npc) => {
				return (
					npc.area === area && Math.abs(npc.x - playerX) < 64 && Math.abs(npc.y - playerY) < 64
				);
			}) || null;

		if (!nearbyNPC) {
			showNPCMessage = false;
		}
	};

	const detectCollisions = () => {
		//prevent player from walking through npcs
		npcs.forEach((npc) => {
			if (npc.area === area && Math.abs(npc.x - playerX) < 34 && Math.abs(npc.y - playerY) < 34) {
				if (keys.w) {
					playerY += playerSpeed;
				}
				if (keys.a) {
					playerX += playerSpeed;
				}
				if (keys.s) {
					playerY -= playerSpeed;
				}
				if (keys.d) {
					playerX -= playerSpeed;
				}
			}
		});
	};
</script>

<svelte:window on:keydown={handleKeyDown} on:keyup={handleKeyUp} />
<main class="rpg">
	{#if ready}
		<div style="color: white">
			{area}
		</div>
		<div class="viewport">
			<div
				class="player character"
				style="height: 32px; width: 32px; background-color: white; position: absolute; top: {playerY}px; left: {playerX}px;"
			></div>
			{#each npcs as npc}
				{#if npc.area === area}
					<div
						class="npc character"
						style="height: 32px; width: 32px; background-color: {npc.background}; position: absolute; top: {npc.y}px; left: {npc.x}px;"
					></div>
					{#if nearbyNPC === npc && npc.message}
						<button
							on:click={() => (showNPCMessage = !showNPCMessage)}
							style="position: absolute; top: {npc.y - 32}px; left: {npc.x + 8}px;"
						>
							!
						</button>
					{/if}
				{/if}
				{#if nearbyNPC === npc && showNPCMessage}
					<div
						transition:fade={{ duration: 50 }}
						class="message"
						style="position: absolute; bottom: 0px; left: 0px; width: 100%; text-align: center;"
					>
						{nearbyNPC.message}
					</div>
				{/if}
			{/each}
		</div>
	{/if}
</main>

<style>
	.rpg {
		height: 100vh;
		width: 100vw;
		background-color: #000;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.viewport {
		height: 500px;
		width: 500px;
		background: white;
		position: relative;
		overflow: hidden;
	}

	.character {
		border: 1px solid black;
	}

	.message {
		border: 1px solid black;
		background-color: white;
		height: 100px;
	}
</style>
