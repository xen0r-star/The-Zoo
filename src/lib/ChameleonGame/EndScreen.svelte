<script lang="ts">
    export let onEnd: () => void;
</script>

<style>
    .screen {
        position: fixed; 
        top: 0; 
        left: 0; 
        width: 100vw; 
        height: 100vh; 
        background: rgba(30,30,30,0.95); 
        z-index: 5; 
        display: flex; 
        flex-direction: column; 
        align-items: center; 
        justify-content: center; 
        gap: 32px;
    }

    .text {
        color: #fff; 
        font-size: 2.5rem;
    }

    .button {
        font-size: 1.2rem; 
        padding: 12px 36px; 
        border-radius: 12px; 
        background: #fff; 
        color: #76912f; 
        font-weight: bold; 
        border: none; 
        cursor: pointer;
    }
</style>

<div class="screen">
    <h1 class="text">
        <slot />
    </h1>
    <button onclick={onEnd} class="button">
        Replay
    </button>
</div>