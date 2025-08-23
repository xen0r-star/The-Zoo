<script lang="ts">
	import { onMount } from 'svelte';
    import { base } from '$app/paths';

	import StartScreen from '$lib/ChameleonGame/StartScreen.svelte';
	import PauseScreen from '$lib/ChameleonGame/PauseScreen.svelte';
	import EndScreen from '$lib/ChameleonGame/EndScreen.svelte';

    import Leaf from '$lib/assets/ChameleonGame/Leaf.svg';
    import BackIcon from '$lib/assets/ChameleonGame/BackIcon.svg';
    import RestartIcon from '$lib/assets/ChameleonGame/RestartIcon.svg';
    import PauseIcon from '$lib/assets/ChameleonGame/PauseIcon.svg';
    import Fly from '$lib/assets/ChameleonGame/Fly.svg';
	import Scoreboard from '$lib/ChameleonGame/Scoreboard.svelte';


    const numberToWin = 15;
    const speedFly = 1;


    // Phrases Chameleon
    const phrases = {
        hello: [
            "Hi! Come play with me.",
            "Hello there! Ready for a challenge?",
            "Hey! Can you beat me?",
        ],
        start: [
            "Show me your skills!",
            `Who will reach ${numberToWin} first?`
        ],
        pause: [
            "Taking a break?",
            "Pause time! Rest your eyes.",
            "Even chameleons need a pause!",
            "Paused! Ready when you are.",
            "Okay, I'm not moving anymore"
        ],
        resume: [
            "Back in action!",
            "Let's go again!"
        ],
        restart: [
            "New game, new luck!",
            "Rematch time!",
            "Restarting the fun!"
        ],
        win: [
            "Wow, you won! Impressive!",
            "You beat the chameleon!",
            "Victory is yours!",
        ],
        lose: [
            "The chameleon wins this time!",
            "Chameleon takes the crown!",
        ]
    };


    // Other Functions
    function randomPhrase(type: keyof typeof phrases) {
        const arr = phrases[type];
        return arr[Math.floor(Math.random() * arr.length)];
    }

    function randomInterval(min: number, max: number): number {
        return Math.random() * (max - min) + min;
    }


    // Chameleon Color
    type ChameleonColor = "green" | "red" | "purple" | "blue" | "yellow";
    let chameleonPalettes = $state<Partial<Record<ChameleonColor, any>>>({});
    let chameleonColor = $state<ChameleonColor>('green');
    let chameleonActiveColor = $derived(chameleonPalettes && chameleonPalettes[chameleonColor]
        ? chameleonPalettes[chameleonColor]
        : {
            color1: "#6E8823",
            color2: "#90AD28",
            color3: "#8EA628",
            color4: "#E3E63B",
            color5: "#B8CC2B",
            color6: "#76821B"
        }
    );

    
    // Sky Color
    let skyColor = $state([]);
    $effect(() => {
        if (skyColor.length > 0) {
            const hour = new Date().getHours();
            document.body.style.setProperty('--sky-tree-fill', skyColor[Math.floor(hour / 2)]);
        }
    });


    // Speech Bubble Chameleon
    let speechBubble = $state({
        text: randomPhrase('hello'),
        display: false
    });

    $effect(() => {
        if (speechBubble.display) {
            setTimeout(() => {
                speechBubble.display = false;
            }, 10000);
        }
    });
    $effect(() => {
        speechBubble.text;
        speechBubble.display = true;
    });

    
    // Chameleon
    let clickTimestamps: number[] = [];
    let colorClickBlocked = $state(false);

    function changeColorChameleon() {
        if (isPauseGame || colorClickBlocked) return;
        const now = Date.now();
        clickTimestamps.push(now);
        clickTimestamps = clickTimestamps.filter(ts => now - ts < 3000);

        if (clickTimestamps.length > 5) {
            colorClickBlocked = true;
            speechBubble.text = "Heeeee! Slow down, <br>I'm not a robot.";
            chameleonColor = 'red';

            setTimeout(() => {
                colorClickBlocked = false;
                clickTimestamps = [];
            }, 10000);
            return;
        }

        const colors = Object.keys(chameleonPalettes) as ChameleonColor[];
        if (colors.length === 0) return;
        const currentIndex = colors.indexOf(chameleonColor);
        const nextIndex = (currentIndex + 1) % colors.length;
        chameleonColor = colors[nextIndex];
    }

    function animateCatch(targetX: number, targetY: number) {
        const startX = tongueStart.x;
        const startY = tongueStart.y;
        const endX = targetX;
        const endY = targetY;
        const duration = 200; // ms

        function animateForward(startTime: number) {
            function step(time: number) {
                const t = Math.min(1, (time - startTime) / (duration / 2));
                tongueEnd.x = startX + (endX - startX) * t;
                tongueEnd.y = startY + (endY - startY) * t;
                if (t < 1) {
                    requestAnimationFrame(step);
                } else {
                    const returnStartX = tongueEnd.x;
                    const returnStartY = tongueEnd.y;
                    const returnStartTime = performance.now();
                    function animateBackward(time2: number) {
                        const t2 = Math.min(1, (time2 - returnStartTime) / (duration / 2));
                        tongueEnd.x = returnStartX + (startX - returnStartX) * t2;
                        tongueEnd.y = returnStartY + (startY - returnStartY) * t2;
                        if (t2 < 1) {
                            requestAnimationFrame(animateBackward);
                        } else {
                            tongueEnd.x = startX;
                            tongueEnd.y = startY;
                        }
                    }
                    requestAnimationFrame(animateBackward);
                }
            }
            requestAnimationFrame(step);
        }
        requestAnimationFrame(animateForward);
    }

    let flyVisible = $state(true);

    let inArea = false;
    function autoChameleonAttack() {
        if (isPauseGame || !flyVisible) {
            setTimeout(autoChameleonAttack, randomInterval(1000, 3000));
            return;
        }

        const fly = document.getElementById('fly');
        if (fly) {
            const rect = fly.getBoundingClientRect();
            const flyX = rect.left + rect.width / 2;
            const flyY = rect.top + rect.height / 2;
            animateCatch(flyX, flyY);
        }
        
        const hour = new Date().getHours();
        const isDay = hour >= 8 && hour < 20;
        const chance = isDay ? 4/10 : 3/10;

        if (Math.random() < chance) {
            catchFly('chameleon');
        }
        setTimeout(autoChameleonAttack, randomInterval(1000, 3000));
    }


    // Fly
    function moveFly(appearFromOutside = false) {
        const fly = document.getElementById('fly');
        const catchZone = document.getElementById('catchZone');
        if (!fly || !catchZone) return;

        const zoneRect = catchZone.getBoundingClientRect();
        const flyWidth = fly.offsetWidth || 50;
        const flyHeight = fly.offsetHeight || 50;

        let lastFlyX = (fly as any).lastFlyX ?? (zoneRect.left + zoneRect.width / 2);
        let lastFlyY = (fly as any).lastFlyY ?? (zoneRect.top + zoneRect.height / 2);

        let flyX, flyY;
        if (appearFromOutside) {
            flyX = zoneRect.left + Math.random() * (zoneRect.width - flyWidth);
            flyY = zoneRect.top - flyHeight - 20;
            
            const targetX = zoneRect.left + Math.random() * (zoneRect.width - flyWidth);
            const targetY = zoneRect.top + Math.random() * (zoneRect.height - flyHeight);

            fly.style.transition = 'none';
            fly.style.left = `${flyX}px`;
            fly.style.top = `${flyY}px`;
            
            void fly.offsetWidth; // forcing reflow

            fly.style.transition = 'left 0.7s cubic-bezier(.5,1.5,.5,1), top 0.7s cubic-bezier(.5,1.5,.5,1)';
            setTimeout(() => {
                fly.style.left = `${targetX}px`;
                fly.style.top = `${targetY}px`;

                (fly as any).lastFlyX = targetX;
                (fly as any).lastFlyY = targetY;
            }, 10);

        } else if (!isPauseGame) {
            let tries = 0;
            let minDistance = 80;
            do {
                flyX = zoneRect.left + Math.random() * (zoneRect.width - flyWidth);
                flyY = zoneRect.top + Math.random() * (zoneRect.height - flyHeight);
                tries++;
            } while (
                Math.abs(flyX - lastFlyX) < minDistance && 
                Math.abs(flyY - lastFlyY) < minDistance && 
                tries < 20
            );

            fly.style.transition = `left ${speedFly}s linear, top ${speedFly}s linear`;
            fly.style.transform = flyX < lastFlyX ? "scaleX(1)" : "scaleX(-1)";
            
            fly.style.left = `${flyX}px`;
            fly.style.top = `${flyY}px`;

            (fly as any).lastFlyX = flyX;
            (fly as any).lastFlyY = flyY;
        }

        setTimeout(() => moveFly(false), Math.random() * 1000 + 500);
    }

    
    // Score
    let playerScore = $state(0);
    let chameleonScore = $state(0);
    let gameEnd = $state(false);
    function catchFly(type: string) {
        if (isPauseGame || !flyVisible) return;

        console.log(type, "caught the fly!");

        if (type === 'player') {
            playerScore += 1;
        } else if (type === 'chameleon') {
            chameleonScore += 1;
        }

        if (playerScore >= numberToWin) {
            isPauseGame = true;
            gameEnd = true;
            speechBubble.text = randomPhrase('win');
            return;

        } else if (chameleonScore >= numberToWin) {
            isPauseGame = true;
            gameEnd = true;
            speechBubble.text = randomPhrase('lose');
            return;
        }

        console.log("Scores:", playerScore, chameleonScore);    

        flyVisible = false;
        setTimeout(() => {
            flyVisible = true;
            moveFly(true);
        }, 700);
    }


    // Game State
    let showStartScreen = $state(true);
    function startGame() {
        showStartScreen = false;
        playerScore = 0;
        chameleonScore = 0;
        isPauseGame = false;
        chameleonColor = 'green';
        speechBubble.text = randomPhrase('start');
    }

    function restartGame() {
        playerScore = 0;
        chameleonScore = 0;
        showStartScreen = true;
        speechBubble.text = randomPhrase('restart');
        chameleonColor = 'green';
    }

    let isPauseGame = $state(true);
    function pauseGame() {
        if (isPauseGame) {
            isPauseGame = false;
            chameleonColor = 'green';
            speechBubble.text = randomPhrase('resume');
        } else {
            isPauseGame = true;
            chameleonColor = 'purple';
            speechBubble.text = randomPhrase('pause'); 
        }
    }

    function endGame() {
        playerScore = 0; 
        chameleonScore = 0; 
        showStartScreen = true; 
        gameEnd = false;
    }


    // On Mount
    let cooldownCatching = $state(Date.now() + randomInterval(10000, 15000));
    let tongueStart = { x: 120, y: 144 };
    let tongueEnd = $state({ x: 120, y: 144 });

    $effect(() => {
        console.log("New time for cooldown:", new Date(cooldownCatching).toLocaleTimeString());
    });

    let eyeOffsetX = $state(0);
    let eyeOffsetY = $state(0);

    let eyeCenterX = 0;
    let eyeCenterY = 0;

    const minX = -3;
    const maxX = 17;
    const minY = -3;
    const maxY = 5;

    function updateTongueStart() {
        const element = document.getElementById('startTongue');
        if (element) {
            const bbox = element.getBoundingClientRect();
            tongueStart.x = bbox.left + bbox.width / 2 + window.scrollX;
            tongueStart.y = bbox.top + bbox.height / 2 + window.scrollY;
            tongueEnd.x = tongueStart.x;
            tongueEnd.y = tongueStart.y;
        }
    }

    function handleMouseMove(event: MouseEvent) {
        if (isPauseGame) return;
        const dx = event.clientX - eyeCenterX;
        const dy = event.clientY - eyeCenterY;
        eyeOffsetX += (Math.max(minX, Math.min(maxX, dx)) - eyeOffsetX) * 0.2;
        eyeOffsetY += (Math.max(minY, Math.min(maxY, dy)) - eyeOffsetY) * 0.2;
    }
    
    function handleCatchZoneMouseMove(event: MouseEvent) {
        if (isPauseGame) return;
        if (Date.now() > cooldownCatching && inArea) {
            const mouseX = event.clientX;
            const mouseY = event.clientY;
            animateCatch(mouseX, mouseY);
            cooldownCatching = Date.now() + randomInterval(10000, 15000);
        }
    }

    function handleCatchZoneEnter() { inArea = true; }
    function handleCatchZoneLeave() { inArea = false; }

    onMount(() => {
        // Fetch data
        fetch(`${base}/skyColor.json`).then(r => r.json()).then(data => { skyColor = data; });
        fetch(`${base}/chameleonPalettes.json`).then(r => r.json()).then(data => { chameleonPalettes = data; });

        // Event listeners
        const eye = document.getElementById('eye-chameleon1');
        if (eye) {
            const rect = eye.getBoundingClientRect();
            eyeCenterX = rect.left + rect.width / 2;
            eyeCenterY = rect.top + rect.height / 2;
        }
        
        const catchZone = document.getElementById('catchZone');
        if (catchZone) {
            catchZone.addEventListener('mousemove', handleCatchZoneMouseMove);
            catchZone.addEventListener('mouseenter', handleCatchZoneEnter);
            catchZone.addEventListener('mouseleave', handleCatchZoneLeave);
        }
        
        moveFly();
        updateTongueStart();
        autoChameleonAttack();
        
        window.addEventListener('mousemove', handleMouseMove);
        window.addEventListener('resize', updateTongueStart);

        return () => {
            window.removeEventListener('resize', updateTongueStart);
            window.removeEventListener('mousemove', handleMouseMove);
            if (catchZone) {
                catchZone.removeEventListener('mousemove', handleCatchZoneMouseMove);
                catchZone.removeEventListener('mouseenter', handleCatchZoneEnter);
                catchZone.removeEventListener('mouseleave', handleCatchZoneLeave);
            }
        };
    });
</script>

<style>
    .leaf {
        width: 100%; 
        height: auto; 
        position: absolute; 
        top: 0; 
        z-index: 3;
    }

    .button-left {
        position: absolute; 
        top: 0; 
        left: 0; 
        transform: rotate(90deg); 
        z-index: 15; 
        margin: 15px; 
        margin-top: 80px;
    }

    .button-right {
        position: absolute; 
        top: 0; 
        right: 0; 
        z-index: 4; 
        margin: 15px; 
        margin-top: 80px; 
        display: flex; 
        gap: 25px; 
        flex-direction: row-reverse;
    }

    #catchZone {
        background: #FF000000; 
        position: absolute; 
        top: 50%; 
        transform: translateY(-50%); 
        width: 60%; 
        height: 35%;
        z-index: 1;
    }

    #chameleonText {
        z-index: 15; 
        transition: all 1s ease-in-out; 
        position: absolute; 
        bottom: 348px; 
        right: 0; 
        margin-right: 140px; 
        background: #f5f5f5; 
        padding: 15px; 
        border-radius: 10px;
    }

    #chameleonText::after {
        content: "";
        position: absolute;
        bottom: -20px;
        left: 30px;
        width: 0;
        height: 0;
        border-right: 25px solid #00000000;
        border-top: 20px solid #f5f5f5;
    }

    #chameleonTongue {
        position:absolute; 
        pointer-events:none; 
        left:0; 
        top:0; 
        width:100vw; 
        height:100vh;
    }

    #PositionChameleon {
        z-index: 10; 
        position: absolute;
        bottom: 0; 
        right: 0; 
        transform: rotate(9deg) translate(7%, -25%); 
        filter: drop-shadow(-5px 5px 0px rgba(0,0,0,0.25));
        
    }

    #PositionChameleon path {
        transition: fill 1s ease-out;
    }

    #fly {
        transition: all 1s linear, transform .1s linear; 
        cursor: pointer; 
        position: absolute; 
        height: 50px; 
        width: auto; 
        z-index: 2;
    }
</style>



<!-- Screen -->
{#if showStartScreen}
    <StartScreen onStart={startGame} />
{/if}

{#if gameEnd}
    <EndScreen onEnd={endGame}>
        {playerScore >= numberToWin ? "Well done, you won!" : "The chameleon won!"}
    </EndScreen>
{/if}

{#if isPauseGame && !showStartScreen && !gameEnd}
    <PauseScreen onPause={pauseGame} />
{/if}


<!-- Game elements -->
<div class="leaf">
    <img src={Leaf} alt="Leaf" class="full"/>
</div>

<Scoreboard {playerScore} {chameleonScore} />


<!-- Button -->
<button onclick={() => window.location.href = '/The-Zoo/'} class="button button-left" aria-label="Back to home">
    <img src={BackIcon} alt="Back to home"/>
</button>

<div class="button-right">
    <button onclick={restartGame} class="button" aria-label="ReStart" >
        <img src={RestartIcon} alt="Restart game"/>
    </button>
    
    <button onclick={pauseGame} class="button" aria-label="Pause">
        <img src={PauseIcon} alt="Pause game"/>
    </button>
</div>


<!-- Chameleon -->
<div id="catchZone"></div>

<div id="chameleonText" style="opacity: {speechBubble.display ? 100 : 0};">
    {@html speechBubble.text}
</div>

<div>
    <svg id="chameleonTongue">
        <line
            x1={tongueStart.x}
            y1={tongueStart.y}
            x2={tongueEnd.x}
            y2={tongueEnd.y}
            stroke="#D16781"
            stroke-width="8"
            stroke-linecap="round"
        />
    </svg>
</div>

<div id="PositionChameleon">
    <svg width="490" height="296" viewBox="0 0 490 296" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M65.6304 253.03C58.9641 257.763 51.5253 264.862 42.8553 273.106C41.965 273.952 40.985 273.037 41.7951 272.113C50.6569 262.011 60.5925 251.731 61.2686 248.763C49.0391 250.794 20.1489 258.312 23.6075 295.699C65.9256 287.775 65.6304 253.03 65.6304 253.03Z" fill="#7E9B24"/>
        <path d="M79.7607 223.802C76.9802 209.974 62.8334 183.051 28.4908 185.977C27.8804 199.011 36.222 225.354 74.471 226.456C68.9512 220.615 60.9331 212.799 54.7199 206.937C53.8355 206.102 54.8835 204.735 55.8839 205.426C65.3833 211.99 72.2959 218.423 79.7607 223.802Z" fill="#7E9B24"/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M302.799 219.893C288.557 222.534 287.54 224.972 286.319 229.117C299.656 229.271 312.805 229.429 325.585 229.59C329.044 223.347 321.11 216.845 321.11 199.247C315.542 200.324 307.781 200.966 299.137 201.241C299.329 209.511 299.912 213.771 302.799 219.893Z" fill={chameleonActiveColor.color1}/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M218.57 165.235C224.538 173.972 234.236 189.617 234.236 196.526C257.619 200.749 275.386 201.996 299.137 201.241C307.781 200.966 315.542 200.324 321.11 199.247C321.901 199.094 322.648 198.932 323.348 198.761C321.72 193.478 324.243 174.582 343.286 163.203C337.114 165.032 323.795 172.143 319.889 185.96C278.3 193.095 245.425 187.992 218.57 165.235Z" fill={chameleonActiveColor.color2}/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M148.176 180.474C148.989 180.474 162.01 180.271 168.114 177.63C169.643 178.687 171.857 180.082 174.624 181.656C180.112 184.777 187.776 188.603 196.597 191.904C201.273 193.654 206.274 195.256 211.449 196.526C202.158 185.283 182.968 163.602 181.745 162.797C178.738 160.818 166.594 179.061 148.176 180.474Z" fill={chameleonActiveColor.color2}/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M148.176 218.267C138.668 222.534 128.034 217.657 122.134 227.828C134.087 227.855 146.628 227.907 159.569 227.98C181.318 214.442 186.911 206.353 196.597 191.904C187.776 188.603 180.112 184.777 174.624 181.656C168.114 196.526 158.145 211.155 148.176 218.267Z" fill={chameleonActiveColor.color1}/>
        <path d="M479.951 259.735C486.766 260.327 487.783 231.695 478.323 231.695C474.387 231.695 428.316 230.956 366.479 230.123C362.41 235.335 350.814 235.132 346.134 254.289C414.257 256.839 476.828 259.463 479.951 259.735Z" fill="#8A5326"/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M195.987 229.646C189.661 231.093 187.849 232.042 186.424 241.227C185.808 245.2 183.929 247.071 181.948 247.338C185.529 247.025 187.577 244.353 188.153 243.056C188.05 237.341 189.577 232.764 195.987 229.646Z" fill={chameleonActiveColor.color3}/>
        <path d="M74.471 226.456C75.4736 227.517 74.8294 229.261 73.377 229.408C49.7621 231.798 11.6347 235.995 6.31379 236.978C0.718222 238.011 -2.42267 244.806 6.31379 249.169C9.67075 250.845 52.2946 246.934 60.2287 247.95C61.2686 248.083 61.4191 248.101 61.2686 248.763C60.5925 251.731 50.6569 262.011 41.7951 272.113C40.985 273.037 41.965 273.952 42.8553 273.106C51.5253 264.862 58.9641 257.763 65.6304 253.03C69.3318 250.402 72.795 248.503 76.0986 247.543C77.6595 247.37 112.488 247.492 137.35 247.91C139.602 247.948 140.714 252.595 138.859 253.873C130.183 259.852 120.58 268.337 119.434 270.3C117.772 273.145 120.248 280.257 124.317 277.209C144.865 254.858 162.362 250.591 170.297 249.575C172.005 249.277 178.639 249.229 188.87 249.37C188.682 248.75 188.542 248.066 188.459 247.323C188.294 245.839 188.177 244.415 188.153 243.056C187.577 244.353 185.529 247.025 181.948 247.338C179.35 247.687 176.578 245.276 176.252 241.227C176.252 235.558 177.674 231.214 180.396 228.111C173.361 228.062 166.409 228.018 159.569 227.98C146.628 227.907 134.087 227.855 122.134 227.828C109.284 227.799 97.1157 227.799 85.8643 227.834C83.7522 226.575 81.7371 225.225 79.7607 223.802C72.2959 218.423 65.3833 211.99 55.8839 205.426C54.8835 204.735 53.8355 206.102 54.7199 206.937C60.9331 212.799 68.9512 220.615 74.471 226.456V226.456Z" fill="#8A5326"/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M346.134 233.913C340.989 236.765 337.774 237.635 335.758 247.323C335.618 248.259 335.099 249.227 334.37 250.056C335.087 249.244 336.506 247.492 336.728 246.653C337.856 241.695 341 238.42 346.134 233.913Z" fill={chameleonActiveColor.color3}/>
        <path d="M325.585 229.59C312.805 229.429 299.656 229.271 286.319 229.117C262.5 228.844 238.082 228.59 214.094 228.379C214.094 228.379 213.28 229.442 206.566 233.303C201.204 236.386 200.869 240.008 198.631 248.136C198.494 248.635 198.338 249.1 198.166 249.529C229.98 250.163 284.044 251.989 336.368 253.926C336.138 251.07 336.259 248.716 336.728 246.653C336.506 247.492 335.087 249.244 334.37 250.056L334.334 250.097C331.885 252.84 327.131 254.015 326.196 247.323C325.346 241.242 326.862 234.177 333.486 229.69C330.869 229.656 328.235 229.623 325.585 229.59Z" fill="#8A5326"/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M249.698 72.1838C258.65 85.5854 261.233 138.801 261.498 140.649C261.764 142.498 264.486 152.003 270.45 141.462C280.451 123.787 279.322 80.7767 273.502 67.763C265.417 68.5338 257.384 70.0159 249.698 72.1838Z" fill={chameleonActiveColor.color4}/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M204.532 96.5576C211.652 107.53 217.959 133.741 219.18 139.024C220.401 144.307 223.453 146.542 229.963 139.024C236.474 131.506 232.608 90.6651 225.284 82.4455C217.818 86.7862 209.984 92.2014 204.532 96.5576Z" fill={chameleonActiveColor.color4}/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M314.314 70.2653C311.618 69.6211 308.859 69.0779 306.054 68.6344C307.296 91.0075 303.613 129.962 301.985 138.658C300.357 147.355 305.792 148.251 310.937 141.462C327.701 124.069 330.672 89.9222 330.061 75.0226C324.82 73.0916 319.606 71.5218 314.314 70.2653Z" fill={chameleonActiveColor.color4}/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M353.662 86.0594C352.026 104.476 345.727 137.398 344.1 141.462C342.472 145.526 343.693 148.574 350 143.697C356.307 138.821 367.687 118.765 373.599 100.012C367.224 94.6419 360.684 90.0875 353.662 86.0594Z" fill={chameleonActiveColor.color4}/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M146.545 104.888C146.545 118.13 135.888 128.865 122.741 128.865C109.594 128.865 98.9371 118.13 98.9371 104.888C98.9371 91.6466 109.594 80.9121 122.741 80.9121C135.888 80.9121 146.545 91.6466 146.545 104.888Z" fill="#FEFEFF"/>
        <path id="eye-chameleon1" transform='translate({eyeOffsetX}, {eyeOffsetY})' d="M109.951 98.9959C107.388 100.256 105.76 104.888 106.655 108.343C107.55 111.594 111.416 113.707 115.322 113.138C118.74 112.609 120.571 110.171 120.571 106.148C120.571 102.938 119.106 100.54 116.34 99.2397C113.817 98.1019 111.904 98.0206 109.951 98.9959Z" fill="#141311"/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} d="M133.158 58.4393C119.567 70.0617 115.742 72.2968 101.094 76.9295C100.37 77.1596 99.6508 77.394 98.9371 77.6325C131.427 71.5404 149.422 71.1876 178.5 113.138C184.622 117.419 187.747 115.483 192.973 107.123C195.333 104.604 200.297 89.4052 202.86 76.6856C204.732 67.4609 205.546 60.9589 205.831 52.7501C206.197 42.3062 204.895 36.8607 201.355 33.8536C198.71 31.5779 196.391 31.0089 190.084 31.0496C181.01 31.0902 172.668 33.4065 161.397 38.9333C151.997 43.566 144.388 48.8082 133.158 58.4393Z" fill={chameleonActiveColor.color6}/>
        <path onclick={changeColorChameleon} style="cursor: pointer;" role="button" tabindex="0" onkeydown={null} fill-rule="evenodd" clip-rule="evenodd" d="M181.745 162.797C182.968 163.602 202.158 185.283 211.449 196.526L212.67 197.745C206.948 206.965 203.577 212.224 196.597 222.128C189.301 222.269 183.82 224.208 180.396 228.111C177.674 231.214 176.252 235.558 176.252 241.227C176.578 245.276 179.35 247.687 181.948 247.338C183.929 247.071 185.808 245.2 186.424 241.227C187.849 232.042 189.661 231.093 195.987 229.646C189.577 232.764 188.05 237.341 188.153 243.056C188.177 244.415 188.294 245.839 188.459 247.323C188.542 248.066 188.682 248.75 188.87 249.37C190.516 254.796 195.816 255.416 198.166 249.529C198.338 249.1 198.494 248.635 198.631 248.136C200.869 240.008 201.204 236.386 206.566 233.303C213.28 229.442 214.094 228.379 214.094 228.379C226.097 215.219 234.236 205.669 234.236 196.526C234.236 189.617 224.538 173.972 218.57 165.235C245.425 187.992 278.3 193.095 319.889 185.96C323.795 172.143 337.114 165.032 343.286 163.203C324.243 174.582 321.72 193.478 323.348 198.761C327.417 208.108 331.237 212.309 345.32 225.582C340.248 226.261 336.38 227.73 333.486 229.69C326.862 234.177 325.346 241.242 326.196 247.323C327.131 254.015 331.885 252.84 334.334 250.097L334.37 250.056C335.099 249.227 335.618 248.259 335.758 247.323C337.774 237.635 340.989 236.765 346.134 233.913C341 238.42 337.856 241.695 336.728 246.653C336.259 248.716 336.138 251.07 336.368 253.926C336.606 256.873 341.862 262.562 346.134 254.289C350.814 235.132 362.41 235.335 366.479 230.123C366.933 225.536 364.607 220.976 357.731 210.749C355.086 206.279 353.051 199.167 351.017 197.542C351.424 196.932 351.451 196.742 351.627 196.323C354.272 190.024 371.362 181.49 381.942 171.737C382.552 195.307 424.27 189.049 438.909 181.896C457.766 172.682 461.456 169.005 466.415 162.797C478.907 149.753 486.76 132.035 489.405 111.066C490.056 105.823 490.056 93.1037 489.405 87.9021C486.923 68.0709 479.558 49.9872 468.165 35.6827C460.515 26.0516 449.936 16.5424 441.309 11.5033C434.229 7.35825 422.917 3.17258 414.901 1.75026C412.785 1.38452 408.025 1.01879 404.322 0.937505C394.556 0.693676 388.493 1.79089 381.698 4.96062C361.027 14.7137 350.936 36.0484 357.69 55.6764C359.562 61.1625 362.329 65.592 366.479 69.8183C374.699 78.1897 384.261 82.0909 395.085 81.4813C409.734 80.6686 419.377 71.8502 419.377 59.2525C419.377 52.019 416.122 46.3704 410.222 43.2819C407.943 42.1034 407.496 42.0222 403.508 41.9815C398.503 41.9815 394.353 43.16 392.359 45.1106C391.341 46.1266 391.219 46.4923 391.423 47.9959C391.789 50.678 393.213 51.572 399.846 53.1975C403.467 54.0915 405.949 57.9927 405.339 61.9346C404.973 64.4541 403.63 66.1609 400.985 67.5426C394.963 70.5904 386.58 68.7211 380.233 62.9099C376.815 59.7808 374.78 56.733 373.234 52.3441C371.606 47.7114 371.484 41.5345 372.868 37.0644C375.268 29.4245 383.285 22.6786 392.888 20.1997C396.916 19.1432 406.438 18.7368 411.443 19.387C419.214 20.4436 428.655 24.5073 435.857 29.9528C439.56 32.7568 445.948 39.0962 449.447 43.4445C458.847 55.0668 465.073 69.8183 466.619 84.0415C467.555 92.5348 466.049 107.53 463.486 115.211C459.661 126.63 451.116 137.886 440.333 145.608C432.886 150.972 425.725 153.207 420.354 151.866C417.953 151.256 415.878 148.98 408.309 138.658C396.037 122.004 385.085 109.685 373.599 100.012C367.687 118.765 356.307 138.821 350 143.697C343.693 148.574 342.472 145.526 344.1 141.462C345.727 137.398 352.026 104.476 353.662 86.0594C351.357 84.7375 349.001 83.4722 346.582 82.2534C340.967 79.4185 335.499 77.0261 330.061 75.0226C330.672 89.9222 327.701 124.069 310.937 141.462C305.792 148.251 300.357 147.355 301.985 138.658C303.613 129.962 307.296 91.0075 306.054 68.6344C295.608 66.9828 284.507 66.7138 273.502 67.763C279.322 80.7767 280.451 123.787 270.45 141.462C264.486 152.003 261.764 142.498 261.498 140.649C261.233 138.801 258.65 85.5854 249.698 72.1838C245.133 73.4715 240.69 75.0011 236.433 76.7673C233.093 78.1467 229.24 80.1453 225.284 82.4455C232.608 90.6651 236.474 131.506 229.963 139.024C223.453 146.542 220.401 144.307 219.18 139.024C217.959 133.741 211.652 107.53 204.532 96.5576C199.885 100.508 197.368 102.825 192.973 107.123C187.747 115.483 184.622 117.419 178.5 113.138C149.422 71.1876 131.427 71.5404 98.9371 77.6325C82.7635 83.0371 69.2594 90.5706 59.7116 99.4834C56.375 102.572 52.5501 107.286 51.7363 109.236C51.1666 110.577 51.2073 110.618 52.7942 112C63.9027 121.631 93.6426 132.928 120.742 138.211C124.323 138.902 129.455 139.512 130.065 139.634C129.997 139.663 129.93 139.693 129.862 139.722C129.766 139.764 129.67 139.805 129.573 139.847L129.565 139.851L129.546 139.859L129.426 139.91C129.374 139.933 129.321 139.956 129.269 139.978C126.398 141.21 123.59 142.307 120.742 143.291C107.028 148.032 92.4089 150.168 65.571 152.231C82.0506 168.689 117.044 182.864 148.176 180.474C166.594 179.061 178.738 160.818 181.745 162.797ZM122.741 128.865C135.888 128.865 146.545 118.13 146.545 104.888C146.545 91.6466 135.888 80.9121 122.741 80.9121C109.594 80.9121 98.9371 91.6466 98.9371 104.888C98.9371 118.13 109.594 128.865 122.741 128.865Z" fill={chameleonActiveColor.color5}/>
        <path id="startTongue" d="M128.392 145H129.392V146H128.392V145Z" fill="#FF000000"/>
    </svg>
</div>


<!-- Fly -->
<button id="fly" onclick={() => flyVisible ? catchFly('player') : null} aria-label="Catch Fly" style="display: {flyVisible ? 'block' : 'none'};">
    <img src={Fly} alt="Fly" class="full"/>
</button>