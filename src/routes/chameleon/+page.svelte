<script lang="ts">
    import ZooBanner from '$lib/Zoo.svelte';
	import { onMount } from 'svelte';
    import { base } from '$app/paths';



    const phrases = {
        hello: [
            "Hi! Come play with me.",
            "Hello there! Ready for a challenge?",
            "Hey! Can you beat me?",
        ],
        start: [
            "Show me your skills!",
            "Who will reach 15 first?"
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

    function randomPhrase(type: keyof typeof phrases) {
        const arr = phrases[type];
        return arr[Math.floor(Math.random() * arr.length)];
    }


    type ChameleonColor = "green" | "red" | "purple" | "blue" | "yellow";
    let chameleonPalettes = $state<Partial<Record<ChameleonColor, any>>>({});
    let chameleonColor = $state<ChameleonColor>('green');
    let chameleonActiveColor = $derived(
        chameleonPalettes && chameleonPalettes[chameleonColor]
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

    
    const hour = new Date().getHours();
    let skyColor = $state([]);
    $effect(() => {
        if (skyColor.length > 0) {
            document.body.style.backgroundColor = skyColor[Math.floor(hour / 2)];
            document.getElementById('skyTree')?.setAttribute('fill', skyColor[Math.floor(hour / 2)]);
        }
    });


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
    })

    
    let clickTimestamps: number[] = [];
    let colorClickBlocked = $state(false);

    function changeColorChameleon() {
        if (isPauseGame) return;
        if (colorClickBlocked) return;

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
            }, 30000);
            return;
        }

        const colors = Object.keys(chameleonPalettes) as ChameleonColor[];
        const currentIndex = colors.indexOf(chameleonColor);
        const nextIndex = (currentIndex + 1) % colors.length;
        chameleonColor = colors[nextIndex];
    }

    function randomInterval(min: number, max: number): number {
        return Math.random() * (max - min) + min;
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

    function autoChameleonAttack() {
        if (isPauseGame || !flyVisible) {
            setTimeout(autoChameleonAttack, 1000);
            return;
        }

        const fly = document.getElementById('fly');
        if (fly) {
            const rect = fly.getBoundingClientRect();
            const flyX = rect.left + rect.width / 2;
            const flyY = rect.top + rect.height / 2;
            animateCatch(flyX, flyY);
        }
        
        const isDay = hour >= 8 && hour < 20;
        const chance = isDay ? 4/10 : 3/10;

        if (Math.random() < 4/10) {
            catchFly('chameleon');
        }
        setTimeout(autoChameleonAttack, randomInterval(1000, 3000));
    }

    let flyVisible = $state(true);

    let playerScore = $state(0);
    let chameleonScore = $state(0);
    let gameEnd = $state(false);
    function catchFly(type: string) {
        if (isPauseGame || !flyVisible) return;

        if (type === 'player') {
            playerScore += 1;
        } else if (type === 'chameleon') {
            chameleonScore += 1;
        }

        if (playerScore >= 15) {
            isPauseGame = true;
            gameEnd = true;
            speechBubble.text = randomPhrase('win');
            return;

        } else if (chameleonScore >= 15) {
            isPauseGame = true;
            gameEnd = true;
            speechBubble.text = randomPhrase('lose');
            return;
        }

        flyVisible = false;
        setTimeout(() => {
            flyVisible = true;
        }, 700);
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

    let showStartScreen = $state(true);
    function startGame() {
        showStartScreen = false;
        playerScore = 0;
        chameleonScore = 0;
        isPauseGame = false;
        chameleonColor = 'green';
        speechBubble.text = randomPhrase('start');
    }



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

    onMount(function() {
        // Recover data ---------------------------------------
        fetch(`${ base }/skyColor.json`)
        .then(response => response.json())
        .then(data => {
            skyColor = data;
            // loaded = true;
        });

        fetch(`${ base }/chameleonPalettes.json`)
        .then(response => response.json())
        .then(data => {
            chameleonPalettes = data;
            // loaded = true;
        });

        // Initialize elements -------------------------
        const element = document.getElementById('startTongue');
        const catchZone = document.getElementById('catchZone');
        const eye = document.getElementById('eye-chameleon1');
        const fly = document.getElementById('fly');
        
        // Tongue --------------------------------------
        const updateTongueStart = () => {
            if (element) {
                const bbox = element.getBoundingClientRect();
                tongueStart.x = bbox.left + bbox.width / 2 + window.scrollX;
                tongueStart.y = bbox.top + bbox.height / 2 + window.scrollY;
                tongueEnd.x = tongueStart.x;
                tongueEnd.y = tongueStart.y;
            }
        };

        updateTongueStart();
        window.addEventListener('resize', updateTongueStart);

        
        let inArea = $state(false);
        
        if (catchZone) {
            catchZone.addEventListener('mousemove', (event: MouseEvent) => {
                if (isPauseGame) return;

                if (Date.now() > cooldownCatching && inArea) {
                    const mouseX = event.clientX;
                    const mouseY = event.clientY;
                    animateCatch(mouseX, mouseY);
                    
                    cooldownCatching = Date.now() + randomInterval(10000, 15000);
                }
            });

            catchZone.addEventListener('mouseenter', () => {
                inArea = true;
            });

            catchZone.addEventListener('mouseleave', () => {
                inArea = false;
            });
        }


        // Eye -----------------------------------------
        if (eye) {
            const rect = eye.getBoundingClientRect();
            eyeCenterX = rect.left + rect.width / 2;
            eyeCenterY = rect.top + rect.height / 2;
        }

        const handleMouseMove = (event: MouseEvent) => {
            if (isPauseGame) return;

            const dx = event.clientX - eyeCenterX;
            const dy = event.clientY - eyeCenterY;

            eyeOffsetX += (Math.max(minX, Math.min(maxX, dx)) - eyeOffsetX) * 0.2;
            eyeOffsetY += (Math.max(minY, Math.min(maxY, dy)) - eyeOffsetY) * 0.2;
        };


        // Fly -----------------------------------------
        let lastFlyX = 0;
        function moveFly() {
            if (!fly || !catchZone) return;

            if (!isPauseGame && (fly || catchZone)) {
                const zoneRect = catchZone.getBoundingClientRect();
                const flyWidth = fly.offsetWidth || 50;
                const flyHeight = fly.offsetHeight || 50;
    
                const flyX = zoneRect.left + Math.random() * (zoneRect.width - flyWidth);
                const flyY = zoneRect.top + Math.random() * (zoneRect.height - flyHeight);
    
                if (flyX < lastFlyX) {
                    fly.style.transform = "scaleX(1)";
                } else {
                    fly.style.transform = "scaleX(-1)";
                }
                lastFlyX = flyX;
                
                console.log(flyX, flyY);

                fly.style.left = `${flyX}px`;
                fly.style.top = `${flyY}px`;
            };

            setTimeout(moveFly, Math.random() * 1000);
        }

        moveFly();


        // Auto attack ---------------------------------
        autoChameleonAttack();

        // Event listeners -----------------------------
        window.addEventListener('mousemove', handleMouseMove);
        return () => {
            window.removeEventListener('resize', updateTongueStart);
            if (catchZone) {
                catchZone.removeEventListener('mousemove', () => {});
                catchZone.removeEventListener('mouseleave', () => {});
            }
        };
    });
</script>

<style>
    path:focus {
        outline: none;
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
</style>


<ZooBanner />

{#if showStartScreen}
    <div style="position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(118,145,47,0.97); z-index: 5; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 32px;">
        <h1 style="color: #fff; font-size: 2.5rem; margin-bottom: 0;">Chameleon VS Legend</h1>
        <div style="color: #f5f5f5; font-size: 1.2rem; background: #1e1e1e80; border-radius: 16px; padding: 24px 32px; max-width: 400px; text-align: center;">
            Hit as many flies as possible.<br>
            <b>First at 15</b> wins the game!
        </div>
        <button onclick={startGame} style="font-size: 1.5rem; padding: 16px 48px; border-radius: 12px; z-index: 15; background: #fff; color: #76912f; font-weight: bold; border: none; cursor: pointer; box-shadow: 0 4px 16px #0002;">
            START
        </button>
    </div>
{/if}

{#if gameEnd}
    <div style=" position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(30,30,30,0.95); z-index: 5; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 32px;">
        <h1 style="color: #fff; font-size: 2.5rem;">
            {playerScore >= 15 ? "Well done, you won!" : "The chameleon won!"}
        </h1>
        <button onclick={() => { playerScore = 0; chameleonScore = 0; showStartScreen = true; gameEnd = false; }} class="button" style="font-size: 1.2rem; padding: 12px 36px; border-radius: 12px; background: #fff; color: #76912f; font-weight: bold; border: none; cursor: pointer;">
            Replay
        </button>
    </div>
{/if}

{#if isPauseGame && !showStartScreen && !gameEnd}
    <div style="position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(30,30,30,0.85); z-index: 5; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 32px;">
        <h1 style="color: #fff; font-size: 2.5rem; margin-bottom: 0;">Pause</h1>
        <button class="button" style="font-size: 1.2rem; padding: 12px 36px; border-radius: 12px; background: #fff; color: #76912f; font-weight: bold; border: none; cursor: pointer;" onclick={pauseGame}>
            To resume
        </button>
    </div>
{/if}


<div id="rotate-warning" style="display: none; height: 100vh; width: 100vw; z-index: 125; position: absolute; top: 0; left: 0; background: #76912f;flex-direction: column; align-items: center; justify-content: flex-start; padding: 20px; padding-top: 100px;">
    <h1 style="color: #ffffff; text-align: center; font-size: 2rem;">Please rotate your device to landscape mode</h1>
    <p style="color: #ffffff; text-align: center; font-size: 1.2rem; margin-block: 25px;">This game is best played in landscape orientation.</p>
    <svg width="256" height="256" viewBox="0 0 256 256" fill="none" xmlns="http://www.w3.org/2000/svg" style="animation: rotate 7.5s infinite ease-in-out;">
        <path d="M181.333 32H74.6668C71.4668 32 69.3335 34.1333 69.3335 37.3333V218.667C69.3335 221.867 71.4668 224 74.6668 224H181.333C184.533 224 186.667 221.867 186.667 218.667V37.3333C186.667 34.1333 184.533 32 181.333 32Z" fill="#C5EFE3"/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M95.7834 56.888C95.6518 58.2698 95.7497 59.5951 96.0406 60.7868C93.1851 60.3231 89.8709 59.4267 88.2281 58.5765C85.0147 57.2519 84.6268 53.8277 85.2488 52.688C85.838 51.6083 87.3662 50.1115 90.1333 51.3065C90.2151 51.4227 90.303 51.538 90.3974 51.6523C91.3661 53.0748 93.5825 55.1747 95.7834 56.888Z" fill="#CDECF8" fill-opacity="0.75"/>
        <path d="M100.38 69.2167C100.975 69.2167 101.457 69.6303 101.457 70.1404C101.457 70.6505 100.975 71.064 100.38 71.064C99.7852 71.064 99.303 70.6505 99.303 70.1404C99.303 69.6303 99.7852 69.2167 100.38 69.2167Z" fill="#1F1E1E"/>
        <path d="M102.004 64.3482C102.004 63.8381 101.521 63.4245 100.927 63.4245C100.332 63.4245 99.8495 63.8381 99.8495 64.3482C99.8495 64.8583 100.332 65.2718 100.927 65.2718C101.521 65.2718 102.004 64.8583 102.004 64.3482Z" fill="#1F1E1E"/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M99.1884 73.806C100.836 74.3071 102.526 73.8051 103.385 72.672C103.576 72.4201 103.726 72.137 103.826 71.8269C104.372 70.1213 103.183 68.2422 101.169 67.6297C99.1581 67.0183 97.0857 67.9003 96.5345 69.6L96.5327 69.6054L96.5316 69.6089C95.9847 71.3144 97.1742 73.1935 99.1884 73.806ZM100.38 69.2167C100.975 69.2167 101.457 69.6303 101.457 70.1404C101.457 70.6505 100.975 71.064 100.38 71.064C99.7852 71.064 99.303 70.6505 99.303 70.1404C99.303 69.6303 99.7852 69.2167 100.38 69.2167Z" fill="#F2EFE0"/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M100.66 60.2616C100.615 60.2652 100.57 60.2696 100.526 60.2746C99.7575 60.3614 99.0645 60.651 98.5058 61.0793C98.2845 61.249 98.0843 61.4403 97.9088 61.6494C97.4883 62.1504 97.2097 62.7533 97.1235 63.4034C97.0926 63.6362 97.0863 63.8751 97.1071 64.1175C97.2746 66.0742 99.1368 67.5202 101.266 67.3473C102.862 67.2177 104.156 66.2158 104.633 64.8982C104.793 64.4576 104.861 63.9817 104.819 63.4914C104.665 61.6826 103.061 60.3101 101.138 60.2496C100.98 60.2447 100.821 60.2485 100.66 60.2616ZM100.927 63.4245C101.521 63.4245 102.004 63.8381 102.004 64.3482C102.004 64.8583 101.521 65.2718 100.927 65.2718C100.332 65.2718 99.8495 64.8583 99.8495 64.3482C99.8495 63.8381 100.332 63.4245 100.927 63.4245Z" fill="#F2EFE0"/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M100.526 60.2746C100.722 60.2303 100.921 60.1963 101.124 60.1732C101.334 60.1493 101.547 60.1371 101.762 60.1371C103.774 60.1371 105.553 61.2035 106.633 62.8371C106.814 62.5788 106.987 62.3075 107.15 62.0242C107.286 61.7871 107.416 61.5417 107.538 61.2885C107.794 60.757 108.016 60.1915 108.2 59.5978C108.293 59.2969 108.376 58.9888 108.449 58.6741C108.502 58.4435 108.549 58.2093 108.59 57.9719C108.743 57.0888 108.803 56.2207 108.776 55.3867C108.766 55.0412 108.74 54.7015 108.701 54.3691C108.661 54.0325 108.607 53.7033 108.539 53.3829C108.472 53.0659 108.391 52.7576 108.298 52.4593C107.536 50.0212 105.925 48.2534 103.82 47.9069C101.984 47.6046 100.137 48.4445 98.6716 50.0311C100.65 53.9808 102.622 59.0687 101.262 59.7499C101.188 59.7871 101.098 59.8073 100.995 59.8116C100.043 59.8509 97.9205 58.5517 95.7834 56.888C95.6518 58.2698 95.7497 59.5951 96.0406 60.7868C96.2293 61.56 96.4992 62.2769 96.8404 62.9164C97.1425 62.4446 97.5023 62.0184 97.9088 61.6494C98.0843 61.4403 98.2845 61.249 98.5058 61.0793C99.0645 60.651 99.7575 60.3614 100.526 60.2746Z" fill="#528088"/>
        <path d="M90.1333 51.3065C90.2151 51.4227 90.303 51.538 90.3974 51.6523C91.3661 53.0748 93.5825 55.1747 95.7834 56.888C96.2402 53.123 97.0021 51.8317 98.6716 50.0311C97.8646 48.4198 97.0567 46.9979 96.4738 46.1419C94.4619 43.1877 92.021 43.9072 90.8153 44.6859C89.6593 45.4325 88.2283 48.6 90.1333 51.3065Z" fill="#DBF4FC" fill-opacity="0.75"/>
        <path d="M101.262 59.7499C102.622 59.0687 100.65 53.9808 98.6716 50.0311C97.0021 51.8317 96.2402 53.123 95.7834 56.888C97.9205 58.5517 100.043 59.8509 100.995 59.8116C101.098 59.8073 101.188 59.7871 101.262 59.7499Z" fill="#B9D7DF"/>
        <path d="M107.15 62.0242C108.752 62.6 110.367 62.9165 111.106 62.353C111.845 61.7894 111.074 61.3041 110.495 61.398C109.916 61.492 108.059 61.5061 107.538 61.2885C107.416 61.5417 107.286 61.7871 107.15 62.0242Z" fill="#676361"/>
        <path d="M108.2 59.5978C109.919 59.8974 111.138 59.7856 111.91 59.5978C112.681 59.4099 112.504 58.0969 111.653 58.408C110.801 58.7192 109.829 58.7273 108.449 58.6741C108.376 58.9888 108.293 59.2969 108.2 59.5978Z" fill="#676361"/>
        <path d="M108.776 55.3867C110.45 55.4394 111.765 55.4023 112.44 54.9484C113.115 54.4944 112.73 53.6334 112.135 53.8682C111.54 54.103 110.175 54.4028 108.701 54.3691C108.74 54.7015 108.766 55.0412 108.776 55.3867Z" fill="#676361"/>
        <path d="M108.539 53.3829C109.982 52.4519 110.833 51.9896 111.299 50.7686C111.765 49.5475 110.431 49.5632 110.254 50.2833C110.077 51.0034 109.244 51.6061 108.298 52.4593C108.391 52.7576 108.472 53.0659 108.539 53.3829Z" fill="#676361"/>
        <path d="M101.124 60.1732C101.334 60.1493 101.547 60.1371 101.762 60.1371C103.774 60.1371 105.553 61.2035 106.633 62.8371C107.322 63.8787 107.726 65.1508 107.726 66.5241C107.726 66.7598 107.714 66.9925 107.691 67.2216C109.015 67.3278 109.768 67.4161 111.122 67.6092C111.399 67.0677 111.887 66.747 112.329 66.8614C112.87 67.0012 113.138 67.7377 112.929 68.5065C112.719 69.2753 112.111 69.7852 111.57 69.6454C111.178 69.544 110.929 69.1287 110.902 68.6148C109.551 68.4638 108.794 68.4375 107.443 68.4739C106.837 70.5034 105.308 72.0911 103.385 72.672C103.576 72.4201 103.726 72.137 103.826 71.8269C104.372 70.1213 103.183 68.2422 101.169 67.6297C99.1581 67.0183 97.0857 67.9003 96.5345 69.6C96.2727 69.0908 96.0739 68.5392 95.9493 67.9573C95.8979 67.717 95.8591 67.4714 95.8338 67.2216C95.8105 66.9925 95.7986 66.7598 95.7986 66.5241C95.7986 65.7398 95.9306 64.9884 96.1722 64.2941C96.2603 64.0409 96.363 63.7953 96.4792 63.5584C96.588 63.3364 96.7088 63.1221 96.8404 62.9164C97.1425 62.4446 97.5023 62.0184 97.9088 61.6494C97.4883 62.1504 97.2097 62.7533 97.1235 63.4034C97.0926 63.6362 97.0863 63.8751 97.1071 64.1175C97.2746 66.0742 99.1368 67.5202 101.266 67.3473C102.862 67.2177 104.156 66.2158 104.633 64.8982C104.793 64.4576 104.861 63.9817 104.819 63.4914C104.665 61.6826 103.061 60.3101 101.138 60.2496C100.98 60.2447 100.821 60.2485 100.66 60.2616C100.615 60.2652 100.57 60.2696 100.526 60.2746C100.722 60.2303 100.921 60.1963 101.124 60.1732Z" fill="#696463"/>
        <path d="M92.2659 69.4132C93.0536 68.0982 95.9493 67.9573 95.9493 67.9573C95.8979 67.717 95.8591 67.4714 95.8338 67.2216C93.0846 67.3964 91.4943 68.2078 90.8192 68.8809C90.144 69.5541 91.4783 70.7282 92.2659 69.4132Z" fill="#484545"/>
        <path d="M92.057 65.0612C93.2305 64.3098 93.3502 64.0371 96.1722 64.2941C96.2603 64.0409 96.363 63.7953 96.4792 63.5584C94.186 63.1833 91.4622 63.3773 90.8192 64.1175C90.1762 64.8577 90.8835 65.8126 92.057 65.0612Z" fill="#484545"/>
        <path d="M176.394 170.942C176.731 170.201 177.13 169.426 177.581 168.615C178.178 167.541 177.272 166.904 176.496 167.857C175.672 168.869 175.02 169.572 174.646 169.736C174.533 165.429 175.195 155.075 188.207 153.78C188.294 168.742 176.394 170.942 176.394 170.942Z" fill="#7E9B24"/>
        <path d="M164.09 158.656C159.179 158.623 149.038 155.58 147.77 143.67C152.186 142.598 161.744 143.7 164.648 156.676C163.977 156.251 163.239 155.768 162.47 155.254C161.459 154.579 160.591 155.613 161.52 156.397C162.414 157.151 163.272 157.888 164.09 158.656Z" fill="#7E9B24"/>
        <path d="M153.033 184.265C153.087 184.543 153.878 188.998 153.378 191.255C153.841 191.707 154.464 192.37 155.185 193.21C156.615 194.875 158.429 197.237 160.141 200.028C161.048 201.507 161.926 203.107 162.702 204.788C158.244 202.363 149.563 197.252 149.207 196.888C148.331 195.993 153.767 190.642 153.033 184.265Z" fill="#22AD78"/>
        <path d="M165.956 181.762C166.787 178.236 164.416 174.931 167.504 172.245C168.303 176.321 169.15 180.596 170.029 185.006C166.838 193.322 164.441 195.766 160.141 200.028C158.429 197.237 156.615 194.875 155.185 193.21C159.839 190.004 164.183 185.634 165.956 181.762Z" fill="#1A855C"/>
        <path d="M173.005 197.32C173.082 195.066 173.287 194.385 176.334 193.291C177.651 192.817 178.167 192.053 178.127 191.359C178.257 192.602 177.478 193.477 177.073 193.759C175.112 194.103 173.648 194.927 173.005 197.32Z" fill="#169A68"/>
        <path d="M122.713 226.071C127.887 228.238 146.254 225.595 146.903 225.563C147.553 225.531 150.983 225.831 147.773 228.563C142.39 233.146 127.609 235.609 122.775 234.485C122.504 231.676 122.48 228.837 122.713 226.071Z" fill="#3FDDA1"/>
        <path d="M128.063 209.048C132.286 210.751 141.665 211.167 143.552 211.234C145.439 211.3 146.405 212.193 144.264 214.912C142.124 217.631 127.904 219.017 124.609 217.063C125.6 214.228 126.934 211.197 128.063 209.048Z" fill="#3FDDA1"/>
        <path d="M110.314 187.223C113.39 181.816 113.901 180.363 114.518 175.059C114.549 174.797 114.581 174.536 114.616 174.277C114.679 185.764 115.747 191.927 132.013 199.07C133.881 200.874 133.425 202.069 130.912 204.405C130.207 205.377 125.338 208.077 121.158 209.794C118.128 211.044 115.958 211.752 113.17 212.392C109.623 213.209 107.675 213.125 106.413 212.117C105.46 211.365 105.113 210.612 104.71 208.457C104.124 205.359 104.365 202.36 105.51 198.148C106.473 194.635 107.763 191.692 110.314 187.223Z" fill="#17865C"/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M133.706 179.005C134.575 183.49 131.608 187.837 127.081 188.714C122.553 189.591 118.178 186.666 117.31 182.181C116.441 177.696 119.407 173.349 123.935 172.472C128.463 171.595 132.837 174.52 133.706 179.005Z" fill="#34C68F"/>
        <path d="M122.648 176.62C122.909 175.662 124.386 174.8 125.626 174.876C126.797 174.967 127.775 176.145 127.838 177.516C127.884 178.717 127.171 179.503 125.795 179.769C124.698 179.982 123.781 179.641 123.153 178.783C122.598 177.998 122.444 177.351 122.648 176.62Z" fill="#141311"/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M127.081 188.714C131.608 187.837 134.575 183.49 133.706 179.005C132.837 174.52 128.463 171.595 123.935 172.472C119.407 173.349 116.441 177.696 117.31 182.181C118.178 186.666 122.553 189.591 127.081 188.714ZM125.626 174.876C124.386 174.8 122.909 175.662 122.648 176.62C122.444 177.351 122.598 177.998 123.153 178.783C123.781 179.641 124.698 179.982 125.795 179.769C127.171 179.503 127.884 178.717 127.838 177.516C127.775 176.145 126.797 174.967 125.626 174.876Z" fill="#FEFEFF"/>
        <path d="M164.648 156.676C165.078 156.948 165.631 156.606 165.585 156.1C164.917 148.752 164.559 143.912 164.544 142.038C164.527 140.06 166.643 138.539 168.712 141.23C169.507 142.264 173.326 165.958 174.3 169.435C174.398 169.786 174.43 169.831 174.646 169.736C175.02 169.572 175.672 168.869 176.496 167.857C177.272 166.904 178.178 167.541 177.581 168.615C177.13 169.426 176.731 170.201 176.394 170.942C175.74 172.379 175.319 173.686 175.209 174.876L179.279 193.586C179.055 193.563 178.812 193.56 178.552 193.581C178.034 193.623 177.54 193.678 177.073 193.759C177.478 193.477 178.257 192.602 178.127 191.359C178.075 190.45 177.067 189.663 175.662 189.82C173.723 190.196 172.332 190.969 171.451 192.103C170.969 189.706 170.495 187.337 170.029 185.006C169.15 180.596 168.303 176.321 167.504 172.245C166.645 167.863 166.279 164.655 165.872 160.472C165.302 159.834 164.708 159.236 164.09 158.656C163.272 157.888 162.414 157.151 161.52 156.397C160.591 155.613 161.459 154.579 162.47 155.254C163.239 155.768 163.977 156.251 164.648 156.676V156.676Z" fill="#8A5326"/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M149.207 196.888C149.563 197.252 158.244 202.363 162.702 204.788L163.2 205.124C165.974 202.562 167.55 201.063 170.475 198.026C170.041 195.528 170.342 193.529 171.451 192.103C172.332 190.969 173.723 190.196 175.662 189.82C177.067 189.663 178.075 190.45 178.127 191.359C178.167 192.053 177.651 192.817 176.334 193.291C173.287 194.385 173.082 195.066 173.005 197.32C173.648 194.927 175.112 194.103 177.073 193.759C177.54 193.678 178.034 193.623 178.552 193.581C178.812 193.56 179.055 193.563 179.279 193.586C181.243 193.788 181.806 195.555 179.948 196.747C179.813 196.834 179.664 196.918 179.502 196.998C176.871 198.3 175.655 198.653 174.955 200.687C174.078 203.233 173.768 203.581 173.768 203.581C170.062 208.548 167.334 211.957 164.208 212.562C161.846 213.02 155.855 210.747 152.474 209.29C160.863 216.011 164.906 225.216 165.977 237.284L123.128 237.284C122.983 236.357 122.865 235.423 122.775 234.485C127.609 235.609 142.39 233.146 147.773 228.563C150.983 225.831 147.553 225.531 146.903 225.563C146.254 225.595 127.887 228.238 122.713 226.071C122.852 224.429 123.082 222.812 123.404 221.242C123.655 220.011 124.084 218.565 124.609 217.063C127.904 219.017 142.124 217.631 144.264 214.912C146.405 212.193 145.439 211.3 143.552 211.234C141.665 211.167 132.286 210.751 128.063 209.048C129.107 207.201 129.733 206.189 130.912 204.405C133.425 202.069 133.881 200.874 132.013 199.07C115.747 191.927 114.679 185.764 114.616 174.277C115.395 168.401 117.079 163.295 119.495 159.448C120.331 158.105 121.69 156.488 122.303 156.081C122.724 155.798 122.741 155.809 123.318 156.259C127.345 159.411 133.173 168.809 136.77 177.704C137.243 178.88 137.79 180.591 137.872 180.791C138.104 179.73 138.358 178.405 138.507 177.368C139.222 172.375 138.986 167.246 137.918 157.954C144.635 162.486 151.793 173.486 153.033 184.265C153.767 190.642 148.331 195.993 149.207 196.888ZM133.706 179.005C134.575 183.49 131.608 187.837 127.081 188.714C122.553 189.591 118.178 186.666 117.31 182.181C116.441 177.696 119.407 173.349 123.935 172.472C128.463 171.595 132.837 174.52 133.706 179.005Z" fill="#34C68F"/>
        <path d="M152.474 209.29C155.855 210.747 161.846 213.02 164.208 212.562C167.197 220.26 168.797 226.239 170.108 234.392C170.269 235.393 170.409 236.363 170.524 237.284L165.977 237.284C164.906 225.216 160.863 216.011 152.474 209.29Z" fill="#22AD78"/>
        <path d="M176.728 234.406C176.69 229.372 177.456 228.864 178.793 228.173C179.425 231.246 180.052 234.289 180.67 237.284L170.524 237.284C170.409 236.363 170.269 235.393 170.108 234.392C172.949 233.91 174.444 233.827 176.728 234.406Z" fill="#1A855C"/>
        <path d="M178.793 228.173C177.126 220.065 175.425 211.751 173.768 203.581C173.768 203.581 174.078 203.233 174.955 200.687C175.655 198.653 176.871 198.3 179.502 196.998C179.664 196.918 179.813 196.834 179.948 196.747L182.768 209.529L189.127 237.284L180.67 237.284C180.052 234.289 179.425 231.246 178.793 228.173Z" fill="#8A5326"/>
        <path d="M138.492 177.316C138.374 178.182 138.229 179.039 138.057 179.903C138.062 179.879 108.735 80.7367 108.085 77.7368C107.436 74.7368 109.935 74.7369 110.277 76.2368C110.619 77.7366 138.492 177.316 138.492 177.316Z" fill="#D16781"/>
        <path d="M181.333 16.0005C193.066 16.0005 202.667 25.6002 202.667 37.3335V218.667C202.667 230.4 193.066 240 181.333 240H74.667C62.9337 240 53.333 230.4 53.333 218.667V37.3335C53.333 25.6002 62.9337 16.0005 74.667 16.0005H181.333ZM74.667 32.0005C71.467 32.0005 69.333 34.1335 69.333 37.3335V218.667C69.333 221.867 71.467 224 74.667 224H181.333C184.533 224 186.667 221.867 186.667 218.667V37.3335C186.667 34.1335 184.533 32.0005 181.333 32.0005H74.667Z" fill="#37474F"/>
    </svg>
</div>

<div style="width: 100%; height: auto; position: absolute; top: 0; z-index: 0;">
    <svg width="1483" height="169" viewBox="0 0 1483 169" fill="none" xmlns="http://www.w3.org/2000/svg" style="width: 100%; height: 100%;">
        <g clip-path="url(#clip0_63_3)">
            <path d="M1293 169C1274.53 169 1257.16 166.065 1241.99 160.898C1234.59 161.915 1226.89 162.451 1219 162.451C1186.67 162.451 1157.68 153.454 1138.06 139.233C1119.42 149.122 1095.32 155.084 1069 155.084C1036.67 155.084 1007.68 146.087 988.062 131.866C969.424 141.755 945.322 147.717 919 147.717C886.666 147.717 857.682 138.72 838.062 124.499C819.424 134.388 795.322 140.35 769 140.35C757.447 140.35 746.321 139.202 735.898 137.077C717.576 146.295 694.314 151.81 669 151.81C644.325 151.81 621.601 146.571 603.5 137.768C585.399 146.571 562.675 151.81 538 151.81C500.532 151.81 467.562 139.729 448.449 121.439C429.188 133.145 402.936 140.35 374 140.35C347.884 140.35 323.952 134.481 305.376 124.73C286.396 143.468 253.013 155.903 215 155.903C186.73 155.903 161.021 149.026 141.893 137.793C121.569 153.192 87.7656 163.27 49.5 163.27C-12.6318 163.27 -63 136.7 -63 103.924C-63 94.7288 -59.0352 86.0219 -51.959 78.2533C-59.0879 71.6205 -63 64.4043 -63 56.8565C-63 44.9292 -53.2314 33.8296 -36.4375 24.5524C-55.4033 9.72607 -67 -10.1331 -67 -31.9578C-67 -46.3569 -61.9521 -59.9005 -53.0703 -71.7076C-68.9531 -82.2142 -79 -97.7318 -79 -115.042C-79 -146.688 -45.4209 -172.342 -4 -172.342C15.7539 -172.342 33.7227 -166.507 47.1182 -156.971C62.3535 -163.62 80.0811 -167.43 99 -167.43C113.747 -167.43 127.77 -165.115 140.454 -160.945C140.154 -162.77 140 -164.624 140 -166.5C140 -195.495 176.713 -219 222 -219C223.542 -219 225.073 -218.973 226.594 -218.919C244.933 -237.282 272.797 -249 304 -249C344.737 -249 379.785 -229.026 395.373 -200.369C406.398 -205.217 419.262 -208 433 -208C464.863 -208 492.022 -193.033 502.445 -172.042C516.587 -177.47 532.574 -180.527 549.5 -180.527C575.479 -180.527 599.244 -173.324 617.524 -161.403C630.418 -187.648 669.406 -206.722 715.5 -206.722C745.519 -206.722 772.522 -198.632 791.269 -185.741C805.88 -190.767 822.657 -193.624 840.5 -193.624C851.902 -193.624 862.869 -192.457 873.113 -190.303C891.481 -207.602 922.867 -219 958.5 -219C1003.11 -219 1041.07 -201.133 1055.15 -176.19C1067.46 -179.012 1080.5 -180.527 1094 -180.527C1110.8 -180.527 1126.89 -178.18 1141.75 -173.887C1156.58 -183.819 1177.42 -190 1200.5 -190C1203.12 -190 1205.71 -189.921 1208.26 -189.765C1222.19 -211.065 1254.66 -226 1292.5 -226C1336.26 -226 1372.85 -206.022 1381.87 -179.326C1398.97 -186.37 1418.4 -190.35 1439 -190.35C1507.48 -190.35 1563 -146.372 1563 -92.1224C1563 -48.4176 1526.97 -11.3794 1477.15 1.36815C1495.54 18.8154 1507 43.4585 1507 70.7721C1507 123.666 1464.02 166.544 1411 166.544C1393.11 166.544 1376.37 161.664 1362.03 153.165C1343.4 163.044 1319.31 169 1293 169Z" fill="#76912F"/>
        </g>
        <defs>
            <clipPath id="clip0_63_3">
                <rect width="1483" height="169" fill="white"/>
            </clipPath>
        </defs>
    </svg>
</div>

<button class="button" aria-label="Back to home" style="cursor: pointer; position: absolute; top: 0; left: 0; transform: rotate(90deg); z-index: 15; margin: 15px; margin-top: 80px;" onclick={() => window.location.href = '/'}>
    <svg width="50" height="50" viewBox="0 0 256 256" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M88 145.792L127.5 189M127.5 189L127.5 67M127.5 189L167 145.792" stroke="#ffffffd1" stroke-width="20" stroke-linecap="round" stroke-linejoin="round"/>
    </svg>
</button>

<div style="position: absolute; top: 0; right: 0; z-index: 4; margin: 15px; margin-top: 80px; display: flex; gap: 25px; flex-direction: row-reverse;">
    <button class="button" aria-label="ReStart" style="cursor: pointer;" onclick={restartGame}>
        <svg width="50" height="50" viewBox="0 0 256 256" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M128.053 67C155.929 67 174.259 76.5382 183.043 95.6143C185.849 101.708 180.77 108 174.062 108V108C169.513 108 165.673 104.946 163.321 101.053C162.202 99.202 160.98 97.6702 159.707 96.3984C154.606 91.3016 145.325 87 128.053 87C110.78 87.0001 101.501 91.3016 96.3994 96.3984C91.3013 101.492 87 110.754 87 128C87 145.246 91.3013 154.508 96.3994 159.602C101.501 164.698 110.78 169 128.053 169C145.325 169 154.606 164.698 159.707 159.602C163.718 155.594 167.235 149.007 168.554 137.982C169.209 132.498 173.584 128 179.106 128V128C184.629 128 189.174 132.499 188.659 137.997C185.473 171.999 165.27 189 128.053 189C87.3509 189 67 168.667 67 128C67 87.3335 87.3509 67.0001 128.053 67Z" fill="#ffffffd1"/>
            <path d="M184.019 72.0835V102.583H153.492" stroke="#ffffffd1" stroke-width="20" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
    </button>
    
    <button class="button" aria-label="Pause" style="cursor: pointer;" onclick={pauseGame}>
        <svg width="50" height="50" viewBox="0 0 256 256" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M159 175L159 80" stroke="#ffffffd1" stroke-width="31" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M96 175L96 80" stroke="#ffffffd1" stroke-width="31" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
    </button>
</div>


<div id="chameleonText" style="opacity: {speechBubble.display ? 100 : 0}; z-index: 15; transition: all 1s ease-in-out; position: absolute; bottom: 348px; right: 0; margin-right: 140px; background: #f5f5f5; padding: 15px; border-radius: 10px;">
    {@html speechBubble.text}
</div>


<div id="catchZone" style="background: #FF000000; position: absolute; top: 50%; transform: translateY(-50%); width: 60%; height: 35%;"></div>

<div>
    <svg id="chameleonTongue" style="position:absolute; pointer-events:none; left:0; top:0; width:100vw; height:100vh;">
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

<div id="PositionChameleon" style="z-index: 10; position: absolute; bottom: 0; right: 0; transform: rotate(9deg) translate(7%, -25%); filter: drop-shadow(-5px 5px 0px rgba(0,0,0,0.25));">
    <svg width="490" height="296" viewBox="0 0 490 296" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path style="transition: fill 1s ease-out;" d="M65.6304 253.03C58.9641 257.763 51.5253 264.862 42.8553 273.106C41.965 273.952 40.985 273.037 41.7951 272.113C50.6569 262.011 60.5925 251.731 61.2686 248.763C49.0391 250.794 20.1489 258.312 23.6075 295.699C65.9256 287.775 65.6304 253.03 65.6304 253.03Z" fill="#7E9B24"/>
        <path style="transition: fill 1s ease-out;" d="M79.7607 223.802C76.9802 209.974 62.8334 183.051 28.4908 185.977C27.8804 199.011 36.222 225.354 74.471 226.456C68.9512 220.615 60.9331 212.799 54.7199 206.937C53.8355 206.102 54.8835 204.735 55.8839 205.426C65.3833 211.99 72.2959 218.423 79.7607 223.802Z" fill="#7E9B24"/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M302.799 219.893C288.557 222.534 287.54 224.972 286.319 229.117C299.656 229.271 312.805 229.429 325.585 229.59C329.044 223.347 321.11 216.845 321.11 199.247C315.542 200.324 307.781 200.966 299.137 201.241C299.329 209.511 299.912 213.771 302.799 219.893Z" fill={chameleonActiveColor.color1}/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M218.57 165.235C224.538 173.972 234.236 189.617 234.236 196.526C257.619 200.749 275.386 201.996 299.137 201.241C307.781 200.966 315.542 200.324 321.11 199.247C321.901 199.094 322.648 198.932 323.348 198.761C321.72 193.478 324.243 174.582 343.286 163.203C337.114 165.032 323.795 172.143 319.889 185.96C278.3 193.095 245.425 187.992 218.57 165.235Z" fill={chameleonActiveColor.color2}/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M148.176 180.474C148.989 180.474 162.01 180.271 168.114 177.63C169.643 178.687 171.857 180.082 174.624 181.656C180.112 184.777 187.776 188.603 196.597 191.904C201.273 193.654 206.274 195.256 211.449 196.526C202.158 185.283 182.968 163.602 181.745 162.797C178.738 160.818 166.594 179.061 148.176 180.474Z" fill={chameleonActiveColor.color2}/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M148.176 218.267C138.668 222.534 128.034 217.657 122.134 227.828C134.087 227.855 146.628 227.907 159.569 227.98C181.318 214.442 186.911 206.353 196.597 191.904C187.776 188.603 180.112 184.777 174.624 181.656C168.114 196.526 158.145 211.155 148.176 218.267Z" fill={chameleonActiveColor.color1}/>
        <path style="transition: fill 1s ease-out;" d="M479.951 259.735C486.766 260.327 487.783 231.695 478.323 231.695C474.387 231.695 428.316 230.956 366.479 230.123C362.41 235.335 350.814 235.132 346.134 254.289C414.257 256.839 476.828 259.463 479.951 259.735Z" fill="#8A5326"/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M195.987 229.646C189.661 231.093 187.849 232.042 186.424 241.227C185.808 245.2 183.929 247.071 181.948 247.338C185.529 247.025 187.577 244.353 188.153 243.056C188.05 237.341 189.577 232.764 195.987 229.646Z" fill={chameleonActiveColor.color3}/>
        <path style="transition: fill 1s ease-out;" d="M74.471 226.456C75.4736 227.517 74.8294 229.261 73.377 229.408C49.7621 231.798 11.6347 235.995 6.31379 236.978C0.718222 238.011 -2.42267 244.806 6.31379 249.169C9.67075 250.845 52.2946 246.934 60.2287 247.95C61.2686 248.083 61.4191 248.101 61.2686 248.763C60.5925 251.731 50.6569 262.011 41.7951 272.113C40.985 273.037 41.965 273.952 42.8553 273.106C51.5253 264.862 58.9641 257.763 65.6304 253.03C69.3318 250.402 72.795 248.503 76.0986 247.543C77.6595 247.37 112.488 247.492 137.35 247.91C139.602 247.948 140.714 252.595 138.859 253.873C130.183 259.852 120.58 268.337 119.434 270.3C117.772 273.145 120.248 280.257 124.317 277.209C144.865 254.858 162.362 250.591 170.297 249.575C172.005 249.277 178.639 249.229 188.87 249.37C188.682 248.75 188.542 248.066 188.459 247.323C188.294 245.839 188.177 244.415 188.153 243.056C187.577 244.353 185.529 247.025 181.948 247.338C179.35 247.687 176.578 245.276 176.252 241.227C176.252 235.558 177.674 231.214 180.396 228.111C173.361 228.062 166.409 228.018 159.569 227.98C146.628 227.907 134.087 227.855 122.134 227.828C109.284 227.799 97.1157 227.799 85.8643 227.834C83.7522 226.575 81.7371 225.225 79.7607 223.802C72.2959 218.423 65.3833 211.99 55.8839 205.426C54.8835 204.735 53.8355 206.102 54.7199 206.937C60.9331 212.799 68.9512 220.615 74.471 226.456V226.456Z" fill="#8A5326"/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M346.134 233.913C340.989 236.765 337.774 237.635 335.758 247.323C335.618 248.259 335.099 249.227 334.37 250.056C335.087 249.244 336.506 247.492 336.728 246.653C337.856 241.695 341 238.42 346.134 233.913Z" fill={chameleonActiveColor.color3}/>
        <path style="transition: fill 1s ease-out;" d="M325.585 229.59C312.805 229.429 299.656 229.271 286.319 229.117C262.5 228.844 238.082 228.59 214.094 228.379C214.094 228.379 213.28 229.442 206.566 233.303C201.204 236.386 200.869 240.008 198.631 248.136C198.494 248.635 198.338 249.1 198.166 249.529C229.98 250.163 284.044 251.989 336.368 253.926C336.138 251.07 336.259 248.716 336.728 246.653C336.506 247.492 335.087 249.244 334.37 250.056L334.334 250.097C331.885 252.84 327.131 254.015 326.196 247.323C325.346 241.242 326.862 234.177 333.486 229.69C330.869 229.656 328.235 229.623 325.585 229.59Z" fill="#8A5326"/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M249.698 72.1838C258.65 85.5854 261.233 138.801 261.498 140.649C261.764 142.498 264.486 152.003 270.45 141.462C280.451 123.787 279.322 80.7767 273.502 67.763C265.417 68.5338 257.384 70.0159 249.698 72.1838Z" fill={chameleonActiveColor.color4}/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M204.532 96.5576C211.652 107.53 217.959 133.741 219.18 139.024C220.401 144.307 223.453 146.542 229.963 139.024C236.474 131.506 232.608 90.6651 225.284 82.4455C217.818 86.7862 209.984 92.2014 204.532 96.5576Z" fill={chameleonActiveColor.color4}/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M314.314 70.2653C311.618 69.6211 308.859 69.0779 306.054 68.6344C307.296 91.0075 303.613 129.962 301.985 138.658C300.357 147.355 305.792 148.251 310.937 141.462C327.701 124.069 330.672 89.9222 330.061 75.0226C324.82 73.0916 319.606 71.5218 314.314 70.2653Z" fill={chameleonActiveColor.color4}/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M353.662 86.0594C352.026 104.476 345.727 137.398 344.1 141.462C342.472 145.526 343.693 148.574 350 143.697C356.307 138.821 367.687 118.765 373.599 100.012C367.224 94.6419 360.684 90.0875 353.662 86.0594Z" fill={chameleonActiveColor.color4}/>
        <path style="transition: fill 1s ease-out;" fill-rule="evenodd" clip-rule="evenodd" d="M146.545 104.888C146.545 118.13 135.888 128.865 122.741 128.865C109.594 128.865 98.9371 118.13 98.9371 104.888C98.9371 91.6466 109.594 80.9121 122.741 80.9121C135.888 80.9121 146.545 91.6466 146.545 104.888Z" fill="#FEFEFF"/>
        <path style="transition: fill 1s ease-out;" id="eye-chameleon1" transform='translate({eyeOffsetX}, {eyeOffsetY})' d="M109.951 98.9959C107.388 100.256 105.76 104.888 106.655 108.343C107.55 111.594 111.416 113.707 115.322 113.138C118.74 112.609 120.571 110.171 120.571 106.148C120.571 102.938 119.106 100.54 116.34 99.2397C113.817 98.1019 111.904 98.0206 109.951 98.9959Z" fill="#141311"/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} d="M133.158 58.4393C119.567 70.0617 115.742 72.2968 101.094 76.9295C100.37 77.1596 99.6508 77.394 98.9371 77.6325C131.427 71.5404 149.422 71.1876 178.5 113.138C184.622 117.419 187.747 115.483 192.973 107.123C195.333 104.604 200.297 89.4052 202.86 76.6856C204.732 67.4609 205.546 60.9589 205.831 52.7501C206.197 42.3062 204.895 36.8607 201.355 33.8536C198.71 31.5779 196.391 31.0089 190.084 31.0496C181.01 31.0902 172.668 33.4065 161.397 38.9333C151.997 43.566 144.388 48.8082 133.158 58.4393Z" fill={chameleonActiveColor.color6}/>
        <path style="transition: fill 1s ease-out; cursor: pointer;" onclick={changeColorChameleon} role="button" tabindex="0" onkeydown={null} fill-rule="evenodd" clip-rule="evenodd" d="M181.745 162.797C182.968 163.602 202.158 185.283 211.449 196.526L212.67 197.745C206.948 206.965 203.577 212.224 196.597 222.128C189.301 222.269 183.82 224.208 180.396 228.111C177.674 231.214 176.252 235.558 176.252 241.227C176.578 245.276 179.35 247.687 181.948 247.338C183.929 247.071 185.808 245.2 186.424 241.227C187.849 232.042 189.661 231.093 195.987 229.646C189.577 232.764 188.05 237.341 188.153 243.056C188.177 244.415 188.294 245.839 188.459 247.323C188.542 248.066 188.682 248.75 188.87 249.37C190.516 254.796 195.816 255.416 198.166 249.529C198.338 249.1 198.494 248.635 198.631 248.136C200.869 240.008 201.204 236.386 206.566 233.303C213.28 229.442 214.094 228.379 214.094 228.379C226.097 215.219 234.236 205.669 234.236 196.526C234.236 189.617 224.538 173.972 218.57 165.235C245.425 187.992 278.3 193.095 319.889 185.96C323.795 172.143 337.114 165.032 343.286 163.203C324.243 174.582 321.72 193.478 323.348 198.761C327.417 208.108 331.237 212.309 345.32 225.582C340.248 226.261 336.38 227.73 333.486 229.69C326.862 234.177 325.346 241.242 326.196 247.323C327.131 254.015 331.885 252.84 334.334 250.097L334.37 250.056C335.099 249.227 335.618 248.259 335.758 247.323C337.774 237.635 340.989 236.765 346.134 233.913C341 238.42 337.856 241.695 336.728 246.653C336.259 248.716 336.138 251.07 336.368 253.926C336.606 256.873 341.862 262.562 346.134 254.289C350.814 235.132 362.41 235.335 366.479 230.123C366.933 225.536 364.607 220.976 357.731 210.749C355.086 206.279 353.051 199.167 351.017 197.542C351.424 196.932 351.451 196.742 351.627 196.323C354.272 190.024 371.362 181.49 381.942 171.737C382.552 195.307 424.27 189.049 438.909 181.896C457.766 172.682 461.456 169.005 466.415 162.797C478.907 149.753 486.76 132.035 489.405 111.066C490.056 105.823 490.056 93.1037 489.405 87.9021C486.923 68.0709 479.558 49.9872 468.165 35.6827C460.515 26.0516 449.936 16.5424 441.309 11.5033C434.229 7.35825 422.917 3.17258 414.901 1.75026C412.785 1.38452 408.025 1.01879 404.322 0.937505C394.556 0.693676 388.493 1.79089 381.698 4.96062C361.027 14.7137 350.936 36.0484 357.69 55.6764C359.562 61.1625 362.329 65.592 366.479 69.8183C374.699 78.1897 384.261 82.0909 395.085 81.4813C409.734 80.6686 419.377 71.8502 419.377 59.2525C419.377 52.019 416.122 46.3704 410.222 43.2819C407.943 42.1034 407.496 42.0222 403.508 41.9815C398.503 41.9815 394.353 43.16 392.359 45.1106C391.341 46.1266 391.219 46.4923 391.423 47.9959C391.789 50.678 393.213 51.572 399.846 53.1975C403.467 54.0915 405.949 57.9927 405.339 61.9346C404.973 64.4541 403.63 66.1609 400.985 67.5426C394.963 70.5904 386.58 68.7211 380.233 62.9099C376.815 59.7808 374.78 56.733 373.234 52.3441C371.606 47.7114 371.484 41.5345 372.868 37.0644C375.268 29.4245 383.285 22.6786 392.888 20.1997C396.916 19.1432 406.438 18.7368 411.443 19.387C419.214 20.4436 428.655 24.5073 435.857 29.9528C439.56 32.7568 445.948 39.0962 449.447 43.4445C458.847 55.0668 465.073 69.8183 466.619 84.0415C467.555 92.5348 466.049 107.53 463.486 115.211C459.661 126.63 451.116 137.886 440.333 145.608C432.886 150.972 425.725 153.207 420.354 151.866C417.953 151.256 415.878 148.98 408.309 138.658C396.037 122.004 385.085 109.685 373.599 100.012C367.687 118.765 356.307 138.821 350 143.697C343.693 148.574 342.472 145.526 344.1 141.462C345.727 137.398 352.026 104.476 353.662 86.0594C351.357 84.7375 349.001 83.4722 346.582 82.2534C340.967 79.4185 335.499 77.0261 330.061 75.0226C330.672 89.9222 327.701 124.069 310.937 141.462C305.792 148.251 300.357 147.355 301.985 138.658C303.613 129.962 307.296 91.0075 306.054 68.6344C295.608 66.9828 284.507 66.7138 273.502 67.763C279.322 80.7767 280.451 123.787 270.45 141.462C264.486 152.003 261.764 142.498 261.498 140.649C261.233 138.801 258.65 85.5854 249.698 72.1838C245.133 73.4715 240.69 75.0011 236.433 76.7673C233.093 78.1467 229.24 80.1453 225.284 82.4455C232.608 90.6651 236.474 131.506 229.963 139.024C223.453 146.542 220.401 144.307 219.18 139.024C217.959 133.741 211.652 107.53 204.532 96.5576C199.885 100.508 197.368 102.825 192.973 107.123C187.747 115.483 184.622 117.419 178.5 113.138C149.422 71.1876 131.427 71.5404 98.9371 77.6325C82.7635 83.0371 69.2594 90.5706 59.7116 99.4834C56.375 102.572 52.5501 107.286 51.7363 109.236C51.1666 110.577 51.2073 110.618 52.7942 112C63.9027 121.631 93.6426 132.928 120.742 138.211C124.323 138.902 129.455 139.512 130.065 139.634C129.997 139.663 129.93 139.693 129.862 139.722C129.766 139.764 129.67 139.805 129.573 139.847L129.565 139.851L129.546 139.859L129.426 139.91C129.374 139.933 129.321 139.956 129.269 139.978C126.398 141.21 123.59 142.307 120.742 143.291C107.028 148.032 92.4089 150.168 65.571 152.231C82.0506 168.689 117.044 182.864 148.176 180.474C166.594 179.061 178.738 160.818 181.745 162.797ZM122.741 128.865C135.888 128.865 146.545 118.13 146.545 104.888C146.545 91.6466 135.888 80.9121 122.741 80.9121C109.594 80.9121 98.9371 91.6466 98.9371 104.888C98.9371 118.13 109.594 128.865 122.741 128.865Z" fill={chameleonActiveColor.color5}/>
        <path id="startTongue" d="M128.392 145H129.392V146H128.392V145Z" fill="#FF000000"/>
    </svg>
</div>


<button id="fly" onclick={() => flyVisible ? catchFly('player') : null} aria-label="Catch Fly" style=" transition: all .75s linear, transform .1s linear; cursor: pointer; position: absolute; height: 50px; width: auto; z-index: 3;">
    <svg width="314" height="286" viewBox="0 0 314 286" fill="none" xmlns="http://www.w3.org/2000/svg" style="height: 100%; width: 100%;">
        <path fill-rule="evenodd" clip-rule="evenodd" d="M178.692 110.455C164.264 109.116 150.425 110.112 137.981 113.07C142.823 84.0331 152.183 50.3308 161.061 33.6257C174.892 0.948043 210.648 -2.9964 222.549 3.32867C233.824 9.32075 249.454 24.8604 236.975 52.9991C235.762 53.8308 234.558 54.725 233.364 55.6846C218.51 65.536 196.583 88.0738 178.692 110.455Z" fill="#CDECF8" fill-opacity="0.75"/>
        <path d="M49.9544 157.198C49.9544 163.247 45.6364 168.15 40.3099 168.15C34.9833 168.15 30.6653 163.247 30.6653 157.198C30.6653 151.149 34.9833 146.246 40.3099 146.246C45.6364 146.246 49.9544 151.149 49.9544 157.198Z" fill="#1F1E1E"/>
        <path d="M100.793 173.708C106.119 173.708 110.437 168.805 110.437 162.756C110.437 156.707 106.119 151.803 100.793 151.803C95.4662 151.803 91.1482 156.707 91.1482 162.756C91.1482 168.805 95.4662 173.708 100.793 173.708Z" fill="#1F1E1E"/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M2.03253 145.08C-3.19986 161.837 2.0418 179.016 13.874 187.755C16.5043 189.698 19.4602 191.224 22.6991 192.235C40.5085 197.796 60.1305 185.7 66.526 165.218C72.9104 144.772 63.7009 123.698 45.9522 118.092L45.896 118.075L45.8594 118.063C28.05 112.502 8.42805 124.598 2.03253 145.08ZM49.9544 157.198C49.9544 163.247 45.6364 168.15 40.3099 168.15C34.9833 168.15 30.6653 163.247 30.6653 157.198C30.6653 151.149 34.9833 146.246 40.3099 146.246C45.6364 146.246 49.9544 151.149 49.9544 157.198Z" fill="#F2EFE0"/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M143.465 160.044C143.427 159.586 143.382 159.131 143.329 158.678C142.423 150.867 139.399 143.821 134.926 138.139C133.155 135.888 131.157 133.852 128.973 132.068C123.742 127.792 117.446 124.959 110.658 124.082C108.227 123.768 105.733 123.704 103.201 123.915C82.7696 125.619 67.6699 144.556 69.4753 166.212C70.8282 182.442 81.291 195.593 95.0489 200.449C99.6497 202.072 104.619 202.768 109.739 202.341C128.627 200.767 142.958 184.465 143.59 164.901C143.642 163.302 143.602 161.681 143.465 160.044ZM110.437 162.756C110.437 168.805 106.119 173.708 100.793 173.708C95.4662 173.708 91.1482 168.805 91.1482 162.756C91.1482 156.707 95.4662 151.803 100.793 151.803C106.119 151.803 110.437 156.707 110.437 162.756Z" fill="#F2EFE0"/>
        <path fill-rule="evenodd" clip-rule="evenodd" d="M143.329 158.678C143.792 160.672 144.146 162.702 144.388 164.763C144.637 166.896 144.765 169.063 144.765 171.256C144.765 191.71 133.63 209.799 116.571 220.783C119.269 222.63 122.102 224.387 125.059 226.043C127.535 227.429 130.099 228.746 132.742 229.985C138.292 232.587 144.197 234.851 150.397 236.72C153.539 237.666 156.756 238.511 160.041 239.247C162.45 239.786 164.895 240.267 167.374 240.686C176.596 242.245 185.661 242.846 194.37 242.581C197.977 242.471 201.524 242.212 204.995 241.812C208.51 241.406 211.948 240.855 215.293 240.165C218.603 239.482 221.823 238.663 224.938 237.715C250.396 229.967 268.856 213.583 272.474 192.183C275.631 173.511 266.861 154.73 250.293 139.825C209.05 159.94 155.921 179.999 148.808 166.171C148.419 165.415 148.209 164.503 148.165 163.45C147.753 153.772 161.32 132.187 178.692 110.455C164.264 109.116 150.425 110.112 137.981 113.07C129.907 114.989 122.421 117.734 115.743 121.204C120.67 124.276 125.12 127.934 128.973 132.068C131.157 133.852 133.155 135.888 134.926 138.139C139.399 143.821 142.423 150.867 143.329 158.678Z" fill="#528088"/>
        <path d="M236.975 52.9991C235.762 53.8308 234.558 54.725 233.364 55.6846C218.51 65.536 196.583 88.0738 178.692 110.455C218.007 115.101 231.491 122.848 250.293 139.825C267.119 131.619 281.967 123.403 290.905 117.475C321.752 97.0169 314.239 72.1949 306.108 59.9345C298.312 48.179 265.237 33.6273 236.975 52.9991Z" fill="#DBF4FC" fill-opacity="0.75"/>
        <path d="M148.808 166.171C155.921 179.999 209.05 159.94 250.293 139.825C231.491 122.848 218.007 115.101 178.692 110.455C161.32 132.187 147.753 153.772 148.165 163.45C148.209 164.503 148.419 165.415 148.808 166.171Z" fill="#B9D7DF"/>
        <path d="M125.059 226.043C119.047 242.329 115.742 258.75 121.627 266.27C127.511 273.789 132.579 265.943 131.598 260.058C130.617 254.173 130.47 235.285 132.742 229.985C130.099 228.746 127.535 227.429 125.059 226.043Z" fill="#676361"/>
        <path d="M150.397 236.72C147.268 254.204 148.435 266.597 150.397 274.443C152.358 282.289 166.069 280.491 162.82 271.828C159.571 263.164 159.486 253.288 160.041 239.247C156.756 238.511 153.539 237.666 150.397 236.72Z" fill="#676361"/>
        <path d="M194.37 242.581C193.819 259.602 194.206 272.972 198.947 279.837C203.687 286.703 212.678 282.78 210.226 276.732C207.774 270.683 204.644 256.804 204.995 241.812C201.524 242.212 197.977 242.471 194.37 242.581Z" fill="#676361"/>
        <path d="M215.293 240.165C225.014 254.842 229.842 263.491 242.592 268.231C255.343 272.972 255.179 259.404 247.66 257.606C240.14 255.808 233.847 247.333 224.938 237.715C221.823 238.663 218.603 239.482 215.293 240.165Z" fill="#676361"/>
        <path d="M144.388 164.763C144.637 166.896 144.765 169.063 144.765 171.256C144.765 191.71 133.63 209.799 116.571 220.783C105.695 227.787 92.4116 231.902 78.0708 231.902C75.6099 231.902 73.1801 231.781 70.7883 231.545C69.6791 245.006 68.7574 252.66 66.7406 266.433C72.3953 269.255 75.7435 274.213 74.5491 278.711C73.0895 284.208 65.3985 286.937 57.3709 284.805C49.3432 282.674 44.0188 276.489 45.4784 270.992C46.5371 267.005 50.8739 264.475 56.2397 264.198C57.8171 250.456 58.0914 242.755 57.7109 229.025C36.5191 222.854 19.9398 207.309 13.874 187.755C16.5043 189.698 19.4602 191.224 22.6991 192.235C40.5085 197.796 60.1305 185.7 66.526 165.218C72.9104 144.772 63.7009 123.698 45.9522 118.092C51.2695 115.431 57.0291 113.409 63.1053 112.142C65.6153 111.619 68.1792 111.225 70.7883 110.967C73.1801 110.731 75.6099 110.61 78.0708 110.61C86.2615 110.61 94.1073 111.952 101.357 114.409C104.001 115.305 106.566 116.349 109.04 117.53C111.358 118.637 113.596 119.865 115.743 121.204C120.67 124.276 125.12 127.934 128.973 132.068C123.742 127.792 117.446 124.959 110.658 124.082C108.227 123.768 105.733 123.704 103.201 123.915C82.7696 125.619 67.6699 144.556 69.4753 166.212C70.8282 182.442 81.291 195.593 95.0489 200.449C99.6497 202.072 104.619 202.768 109.739 202.341C128.627 200.767 142.958 184.465 143.59 164.901C143.642 163.302 143.602 161.681 143.465 160.044C143.427 159.586 143.382 159.131 143.329 158.678C143.792 160.672 144.146 162.702 144.388 164.763Z" fill="#696463"/>
        <path d="M47.9029 74.686C61.6341 82.6959 63.1053 112.142 63.1053 112.142C65.6153 111.619 68.1792 111.225 70.7883 110.967C68.9626 83.011 60.4899 66.8396 53.4608 59.974C46.4317 53.1084 34.1716 66.6761 47.9029 74.686Z" fill="#484545"/>
        <path d="M93.3468 72.561C101.193 84.4941 104.041 85.712 101.357 114.409C104.001 115.305 106.566 116.349 109.04 117.53C112.956 94.2108 110.931 66.5127 103.201 59.974C95.4719 53.4353 85.5004 60.6279 93.3468 72.561Z" fill="#484545"/>
    </svg>
</button>

<div id="scoreboard" style="position: absolute; top: 0; width: 100%; z-index: 1; padding: 75px 0; color: #1e1e1e; font-size: 1.25em; font-weight: bold;">
    <div style="display: flex; flex-direction: row; align-items: center; justify-content: center; max-width: 512px; width: 32%; margin: auto; background: #ebf8ea; border-radius: 25px; padding-block: 4px;">
        <div style="flex: 1; text-align: right;">The legend: {playerScore.toString().padStart(2, '0')}</div>
        <div style="margin-inline: 10px"> - </div>
        <div style="flex: 1; text-align: left;">{chameleonScore.toString().padStart(2, '0')} :Chameleon</div>
    </div>
    <div style="text-align: center; font-size: 0.70em; max-width: 400px; background: #d0d2d0; margin: auto; width: 25%; border-radius: 25px; margin-top: 10px; color: #1e1e1eb0;">Hit the fly as many times as you can!</div>
</div>