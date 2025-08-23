<script lang="ts">
    export let onStart: () => void;
</script>

<style>
    .screen {
        position: fixed; 
        top: 0; 
        left: 0; 
        width: 100vw; 
        height: 100vh; 
        background: rgba(118,145,47,0.97); 
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
        font-size: 1.5rem; 
        padding: 16px 48px; 
        border-radius: 12px; 
        z-index: 15; 
        background: #fff; 
        color: #76912f; 
        font-weight: bold; 
        border: none;
        cursor: pointer; 
        box-shadow: 0 4px 16px #0002;
    }

    .content {
        color: #f5f5f5; 
        font-size: 1.2rem; 
        background: #1e1e1e80; 
        border-radius: 16px; 
        padding: 24px 32px; 
        max-width: 400px; 
        text-align: center;
    }
</style>

<div class="screen">
    <h1 class="text">Chameleon VS Legend</h1>
    <div class="content">
        Hit as many flies as possible.<br>
        <b>First at 15</b> wins the game!
    </div>
    <button onclick={onStart} class="button">
        START
    </button>
</div>