<script lang="ts">
    import { onMount } from 'svelte';
    import { base } from '$app/paths';
    
    import ZooBanner from '$lib/Zoo.svelte';
	import LoadingScreen from '$lib/LoadingScreen.svelte';
	import ProgressBar from '$lib/ProgressBar.svelte';
    
	import TreeAnimation from '$lib/HomePage/TreeAnimation.svelte';
	import Section1 from '$lib/HomePage/Section1.svelte';
	import Section2 from '$lib/HomePage/Section2.svelte';
	import Section3 from '$lib/HomePage/Section3.svelte';
	import Section4 from '$lib/HomePage/Section4.svelte';
	import Section5 from '$lib/HomePage/Section5.svelte';
	import Section6 from '$lib/HomePage/Section6.svelte';
	import Section7 from '$lib/HomePage/Section7.svelte';


    // Loading State
    let loaded = $state({        
        chameleonPalettes: false,
        skyColor: false,
        facts: false
    });
    let allLoaded = $derived(
        loaded.chameleonPalettes && loaded.skyColor && loaded.facts
    ) 


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

    
    // Facts Slider
    interface Fact {
        title?: string;
        icon?: string;
        description?: string;
        chameleonColor?: ChameleonColor;
        background?: string;
        [key: string]: any;
    }
    let facts = $state<Fact[]>([]);
    let currentFacts = $state(0);
    let unlockLevel = $state(0);

    $effect(() => {
        if (loaded.facts) {
            const level = Math.round((currentFacts + 1) / facts.length * 100);
            if (level > unlockLevel) {
                unlockLevel = level;
            }

        } else {
            unlockLevel = 0;
        }
    });

    function syncChameleonColorIndex() {
        chameleonColor = (facts[currentFacts]?.chameleonColor as ChameleonColor) || 'green';
    }
    function nextSlide() {
        if (currentFacts < facts.length - 1) currentFacts += 1;
        setTimeout(syncChameleonColorIndex, 500);
    }
    function prevSlide() {
        if (currentFacts > 0) currentFacts -= 1;
        setTimeout(syncChameleonColorIndex, 500);
    }

    function interactionPage() {
        if (unlockLevel >= 100) {
            window.location.href = 'chameleon';
        }
    }


    // Easter Egg
    let showHeart = $state(false);
    function love() {
        showHeart = false;
        setTimeout(() => {
            showHeart = true;
            setTimeout(() => {
                showHeart = false;
            }, 900);
        }, 10);
    }


    // On Mount
    let scrollProgress = $state(0);
    let eyeOffsetX = $state(0);
    let eyeOffsetY = $state(0);
    let eyeCenterX = 0;
    let eyeCenterY = 0;
    const minX = -3, maxX = 17, minY = -3, maxY = 5;

    function handleMouseMove(event: MouseEvent) {
        const dx = event.clientX - eyeCenterX;
        const dy = event.clientY - eyeCenterY;
        eyeOffsetX += (Math.max(minX, Math.min(maxX, dx)) - eyeOffsetX) * 0.2;
        eyeOffsetY += (Math.max(minY, Math.min(maxY, dy)) - eyeOffsetY) * 0.2;
    }

    function handleScroll() {
        const scrollTop = window.scrollY;
        const docHeight = document.documentElement.scrollHeight - window.innerHeight;
        scrollProgress = docHeight > 0 ? (scrollTop / docHeight) * 100 : 0;
    }

    onMount(() => {
        // Fetch data
        fetch(`${base}/facts.json`).then(r => r.json()).then(data => { facts = data; loaded.facts = true; });
        fetch(`${base}/skyColor.json`).then(r => r.json()).then(data => { skyColor = data; loaded.skyColor = true; });
        fetch(`${base}/chameleonPalettes.json`).then(r => r.json()).then(data => { chameleonPalettes = data; loaded.chameleonPalettes = true; });

        // Eye center
        const eye = document.getElementById('eye-chameleon1');
        if (eye) {
            const rect = eye.getBoundingClientRect();
            eyeCenterX = rect.left + rect.width / 2;
            eyeCenterY = rect.top + rect.height / 2;
        }

        // Event listeners
        window.addEventListener('scroll', handleScroll);
        window.addEventListener('mousemove', handleMouseMove);
        handleScroll();

        return () => {
            window.removeEventListener('scroll', handleScroll);
            window.removeEventListener('mousemove', handleMouseMove);
        };
    });
</script>


<!-- Header -->
<ZooBanner />

<!-- Top Page -->
<TreeAnimation onLove={love} {showHeart} />

<!-- Section Contained -->
<Section1 />
<Section2 />
<Section3 />
<Section4 />
<Section5 {facts} {currentFacts} {chameleonActiveColor} {prevSlide} {nextSlide} {eyeOffsetX} {eyeOffsetY} />
<Section6 onClick={interactionPage} {unlockLevel} />
<Section7 />

<!-- Other Element -->
<LoadingScreen showLoading={!allLoaded} />
<!-- <ProgressBar {scrollProgress} /> -->