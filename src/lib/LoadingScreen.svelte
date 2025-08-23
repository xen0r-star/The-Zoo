<script lang="ts">
    const { showLoading } = $props<{ showLoading: boolean }>();

    let loadingDots = $state('');
    let loadingInterval: any;
    let visible = $state(true);
    let isHiding = $state(false);

    $effect(() => {
        if (showLoading) {
            visible = true;
            isHiding = false;
            loadingInterval = setInterval(() => {
                loadingDots = loadingDots.length < 3 ? loadingDots + '.' : '';
            }, 500);
            
        } else {
            isHiding = true;
            clearInterval(loadingInterval);

            setTimeout(() => {
                visible = false;
            }, 1000);
        }
    });
</script>

<style>
    #loadedScreen {
        color: var(--white-text); 
        display: flex; 
        padding-top: 100px; 
        transition: all 1.5s ease-in-out;
        height: 100vh; 
        width: 100vw; 
        z-index: 125; 
        position: fixed; 
        top: 0; 
        left: 0; 
        background: var(--primary); 
        flex-direction: column; 
        align-items: center; 
        justify-content: flex-start; 
        padding: 20px; 
        padding-top: 180px;
    }

    .icon {
        font-size: 6rem; 
        margin-bottom: 25px;
    }

    h1 {
        text-align: center; 
        font-size: 2rem; 
    }

    p {
        text-align: center; 
        font-size: 1.2rem; 
        margin-block: 5px;
    }
</style>



{#if visible}
    <div id="loadedScreen" style={isHiding ? 'animation: 1s ease-in-out down forwards;' : ''}>
        <span class="material-symbols-outlined bannerIcon icon">hourglass</span>
        <h1>Loading{loadingDots}</h1>
        <p>Please wait while we prepare the chameleon experience for you.</p>
    </div>
{/if}