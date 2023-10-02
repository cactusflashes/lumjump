<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css" />

<style>
    @import url('../lib/styles/style.css');
</style>

<script>
import { onMount } from 'svelte'
import lumModel from '$lib/assets/lum.gif'
import { fly, slide } from 'svelte/transition';
import ataruModel from '$lib/assets/ataru.gif';

import octo from '$lib/assets/oct.gif';
import star from '$lib/assets/star.gif';
import ten from '$lib/assets/ten.gif'; 
import rem from '$lib/assets/rem.gif';
import ghost from '$lib/assets/ghost.png';

let lum; 
let ataru;
 
let lumY = 300;
let velocity = 0;
let gravity = .3;
let jumpStrength = -10;
let seconds = 0; 
let intervalID; 

let gameState = 'start';
let obstacle; 


const obsg = [
    octo,
    star,
    ten,
    rem
];

let previousObsIndex = -1; 

function randObs() {
    let randomIndex;

    do {
        randomIndex = Math.floor(Math.random() * obsg.length);
    } while (randomIndex === previousObsIndex);

    previousObsIndex = randomIndex;

    const randomObsg = obsg[randomIndex];
    return randomObsg; 
}

let src = ghost;

function generateObs() {
    if (gameState === "playing") {
        src = randObs(); 
       const randP = Math.random() * -35;
       obstacle.style.top = `${randP}vh`
    } else if (gameState === 'start' || gameState === 'over') {
        src = ghost; 
    } 
}


setInterval(generateObs, 3200);

function startGame() {
    
    gameState = 'playing';
    obstacle.classList.add('scroll-enabled');
    obstacleTwo.classList.add('scroll-enabled-2'); 
    lum.classList.remove('animate__animated'); // Remove lum's flying animation in start mode
    lum.style.display = 'flex';
    ataru.style.display = 'flex';
    lumY = 300;
    velocity = 0;
    startTimer();
    generateObs();
    randObs();
    update();
}


function startTimer() {
    intervalID = setInterval(function() {
        seconds++;
    }, 1000);
}

function stopTimer() {
    clearInterval(intervalID);
}

function jump() {
    if (gameState === 'playing') {
        velocity = jumpStrength;
    }
}

function checkCollision() {
    if (gameState === 'playing') {
        const obstacleRect = obstacle.getBoundingClientRect();
        const lumRect = lum.getBoundingClientRect();

        const buffer = 50;

        if (
            lumRect.right > obstacleRect.left + buffer &&
            lumRect.left < obstacleRect.right - buffer &&
            lumRect.bottom > obstacleRect.top + buffer &&
            lumRect.top < obstacleRect.bottom - buffer
        ) {
            gameState = 'over';
            stopTimer();
            lum.style.display = 'none';
            ataru.style.display = 'none';
        }
    }
}



function update() {
    if (gameState === 'playing') {
        velocity += gravity;
    }

    lumY += velocity;

    if (lumY < 0) {
        lumY = 0;
    }

    if (lumY > 600) {
        gameState = 'over';
        stopTimer();
        lum.style.display = "none";
        ataru.style.display = "none";
    }

    requestAnimationFrame(update);
    checkCollision();
}


    function resetGame() {
        location.reload(); 
}

onMount(() => {
    gameState = 'start'; 
});

    
</script>

<body>


 <!-- svelte-ignore a11y-click-events-have-key-events -->
 <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div class="container" on:click={jump}>

                {#if gameState === 'start'}

                <div class="start-game" on:click={startGame}></div>
                <h1 class="animate__animated animate__pulse animate__fast animate__infinite" transition:fly={{y: -50}}>Tap to Start</h1>

                {/if}

        <h2>Score: {seconds}</h2>
        <!-- svelte-ignore empty-block -->
        <img class="lum animate__animated animate__shakeY animate__infinite animate__slower" src={lumModel} style="top: {lumY}px;" alt="lum" bind:this={lum}/> 

        <div class="ground" on:click={jump}>  

            <img src={ataruModel} alt="ataru" class="ataru" bind:this={ataru}/>
            
        </div>

        <!-- svelte-ignore a11y-missing-attribute -->
        <img class="obs" {src} alt=" " bind:this={obstacle}/>



                {#if gameState === 'over'}

                    <div class="game-over" transition:slide={{y: 800}}>

                        <h3>GAME OVER</h3>
                        <h3>Your Score: {seconds}</h3>
                        <button class="again-btn" on:click={resetGame}>PLAY AGAIN</button>
                
                    </div>

                {/if}


    </div>

</body>



