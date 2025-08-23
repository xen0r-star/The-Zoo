<script lang="ts">
    import ZooBanner from '$lib/Zoo.svelte';
    import RotateScreen from '$lib/assets/ChameleonGame/RotateScreen.svg';
    import ChameleonGame from '$lib/ChameleonGame.svelte';
</script>

<style>
    :global(body) {
        overflow-y: hidden;
    }

    #rotate-warning {
        display: none; 
        height: 100vh; 
        width: 100vw; 
        z-index: 125; 
        position: absolute; 
        top: 0; left: 0; 
        background: #76912f;
        flex-direction: column; 
        align-items: center; 
        justify-content: flex-start; 
        padding: 20px; 
        padding-top: 100px;
    }

    #rotate-warning > h1 {
        color: #ffffff; 
        text-align: center;
        font-size: 2rem;
    }

    #rotate-warning > p {
        color: #ffffff; 
        text-align: center; 
        font-size: 1.2rem; 
        margin-block: 25px;
    }

    #rotate-warning > img {
        animation: rotate 7.5s infinite ease-in-out;
    }
</style>


<!-- Header -->
<ZooBanner />

<!-- Information -->
<div id="rotate-warning">
    <h1>Please rotate your device to landscape mode</h1>
    <p>This game is best played in landscape orientation.</p>
    <img src={RotateScreen} alt="Rotate your device"/>
</div>

<!-- Game -->
<ChameleonGame />