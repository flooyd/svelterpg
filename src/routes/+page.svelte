<script>
	import { onMount } from 'svelte';

	let playerX = 0;
	let playerY = 0;
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
			area: 0
		});
		npcs.push({
			x: 100,
			y: 100,
			area: 1
		});
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
        if(event.key === 'e') {
            if(nearbyNPC) {
                showNPCMessage = true;
            }
        }
	};

	onMount(() => {
		const gameLoop = () => {
			move();
			requestAnimationFrame(gameLoop);
            checkForNearbyNPCs();
		};

		addEnemies();
		gameLoop();
        
		ready = true;
	});

	//if player goes offscreen, side scroll
	const sideScroll = () => {
		if (playerX > 500) {
			area += 1;
			playerX = 0;
		}
		if (playerX < 0 && area > 0) {
			area -= 1;
			playerX = 500;
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

	const checkForNearbyNPCs = () => {
		nearbyNPC = npcs.find((npc) => {
            return (
                npc.area === area &&
                npc.x > playerX - 64 &&
                npc.x < playerX + 64 &&
                npc.y > playerY - 64 &&
                npc.y < playerY + 64
            );
        });

        if(!nearbyNPC) {
            showNPCMessage = false;
        }
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
				class="player"
				style="height: 32px; width: 32px; background-color: black; position: absolute; top: {playerY}px; left: {playerX}px;"
			></div>
			{#each npcs as npc}
				{#if npc.area === area}
					<div
						class="npc"
						style="height: 32px; width: 32px; background-color: red; position: absolute; top: {npc.y}px; left: {npc.x}px;"
					></div>
                    {#if nearbyNPC === npc && showNPCMessage}
                        <div
                            class="message"
                            style="position: absolute; top: {npc.y - 32}px; left: {npc.x}px;"
                        >
                            Hello friend
                    </div>
                    {/if}
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
	}
</style>
