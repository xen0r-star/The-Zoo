<script lang="ts">
    import ZooBanner from '$lib/Zoo.svelte';
	import { onMount } from 'svelte';

    const skyColor = [
        '#1A1A2E', // 02h
        '#1B263B', // 04h
        '#FFD6D1', // 06h
        '#B3E5FC', // 08h
        '#81D4FA', // 10h
        '#81D4FA', // 12h
        '#81D4FA', // 14h
        '#81D4FA', // 16h
        '#81D4FA', // 18h
        '#334b63', // 20h
        '#1B263B', // 22h
        '#1A1A2E'  // 00h 
    ];

    const chameleonPalettes = {
        green: {
            color1: '#6E8823',
            color2: '#90AD28',
            color3: '#8EA628',
            color4: '#E3E63B',
            color5: '#B8CC2B',
            color6: '#76821B'
        },
        red: {
            color1: '#882325',
            color2: '#AD282A',
            color3: '#A6282A',
            color4: '#E63B3E',
            color5: '#CC2B2E',
            color6: '#821B1D'
        },
        purple: {
            color1: '#88237E',
            color2: '#AD28A0',
            color3: '#A62899',
            color4: '#E63BD5',
            color5: '#CC2BBC',
            color6: '#821B78'
        },
        blue: {
            color1: '#235988',
            color2: '#286FAD',
            color3: '#286BA6',
            color4: '#3B96E6',
            color5: '#2B81CC',
            color6: '#1B5282'
        },
        yellow: {
            color1: '#6E8823',
            color2: '#90AD28',
            color3: '#8EA628',
            color4: '#E3E63B',
            color5: '#B8CC2B',
            color6: '#76821B'
        }
    };


    const hour = new Date().getHours();
    $effect(() => {
        document.body.style.backgroundColor = skyColor[Math.floor(hour / 2)]
    });


    let speechBubble = $state({
        text: "Hi! Come play with me.",
        display: true
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


    type ChameleonColor = "green" | "red" | "purple" | "blue" | "yellow";
    let chameleonColor = $state<ChameleonColor>('green');

	let chameleonActiveColor = $derived(
        chameleonColor ? chameleonPalettes[chameleonColor] : chameleonPalettes.green
    );
    
    let clickTimestamps: number[] = [];
    let colorClickBlocked = $state(false);

    function changeColorChameleon() {
        if (colorClickBlocked) {
            return;
        }
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

    let playerScore = $state(0);
    let chameleonScore = $state(0);
    function catchFly(type: string) {
        if (type === 'player') {
            playerScore += 1;
        } else if (type === 'chameleon') {
            chameleonScore += 1;
        }
    }

    function restartGame() {
        playerScore = 0;
        chameleonScore = 0;
        chameleonColor = 'green';
        speechBubble.text = "Okay, let's start again";
    }

    let isPauseGame = $state(false);
    function pauseGame() {
        if (isPauseGame) {
            isPauseGame = false;
            chameleonColor = 'green';
            speechBubble.text = "Okay, I'm back!";
        } else {
            isPauseGame = true;
            chameleonColor = 'purple';
            speechBubble.text = "Okay, I'm not moving anymore";
        }
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

            if (!isPauseGame) {
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
    
                fly.style.left = `${flyX}px`;
                fly.style.top = `${flyY}px`;
            };

            setTimeout(moveFly, Math.random() * 1500);
        }

        moveFly();


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

<button class="button" aria-label="Back to home" style="cursor: pointer; position: absolute; top: 0; left: 0; transform: rotate(90deg); z-index: 100; margin: 15px; margin-top: 80px;" onclick={() => window.location.href = '/'}>
    <svg width="50" height="50" viewBox="0 0 256 256" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M88 145.792L127.5 189M127.5 189L127.5 67M127.5 189L167 145.792" stroke="#ffffffd1" stroke-width="20" stroke-linecap="round" stroke-linejoin="round"/>
    </svg>
</button>

<div style="position: absolute; top: 0; right: 0; z-index: 100; margin: 15px; margin-top: 80px; display: flex; gap: 25px; flex-direction: row-reverse;">
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



<div id="chameleonText" style="opacity: {speechBubble.display ? 100 : 0}; transition: all 1s ease-in-out; position: absolute; top: 35%; right: 0; margin-right: 25px; background: #f5f5f5; padding: 25px; border-radius: 10px;">
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

<button id="fly" onclick={() => catchFly('player')} aria-label="Catch Fly" style="transition: all .75s linear, transform .1s linear; cursor: pointer; position: absolute; height: 50px; width: auto; z-index: 20;">
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

<div style="position: absolute; top: 0; width: 100%; z-index: 1; padding: 75px 0; color: #1e1e1e; font-size: 1.5em; font-weight: bold;">
    <div style="display: flex; flex-direction: row; align-items: center; justify-content: center; width: 32%; margin: auto; background: #ebf8ea; border-radius: 25px; padding-block: 4px;">
        <div style="flex: 1; text-align: right;">The legend: {playerScore.toString().padStart(2, '0')}</div>
        <div style="margin-inline: 10px"> - </div>
        <div style="flex: 1; text-align: left;">{chameleonScore.toString().padStart(2, '0')} :Chameleon</div>
    </div>
    <div style="text-align: center; font-size: 1.25rem; background: #d0d2d0; margin: auto; width: 25%; border-radius: 25px; margin-top: 10px; color: #1e1e1eb0;">Catch the most flies!</div>
</div>