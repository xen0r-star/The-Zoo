<script lang="ts">
    export let playerScore: number;
    export let chameleonScore: number;
</script>

<style>
    .screen {
        position: absolute; 
        top: 0; 
        width: 100%; 
        z-index: 1; 
        padding: 75px 0; 
        color: #1e1e1e; 
        font-size: 1.25em; 
        font-weight: bold;
    }

    #scoreboard {
        display: flex; 
        flex-direction: row; 
        align-items: center; 
        justify-content: center; 
        max-width: 512px; 
        width: 32%; 
        margin: auto; 
        background: #ebf8ea; 
        border-radius: 25px;
        padding-block: 4px;
    }

    #scoreboard > #player {
        flex: 1; 
        text-align: right;
    }

    #scoreboard > #separator {
        margin-inline: 10px
    }

    #scoreboard > #chameleon {
        flex: 1; 
        text-align: left;
    }

    .text {
        text-align: center; 
        font-size: 0.70em; 
        max-width: 400px; 
        background: #d0d2d0; 
        margin: auto; 
        width: 25%; 
        border-radius: 25px; 
        margin-top: 10px; 
        color: #1e1e1eb0;
    }
</style>

<div class="screen">
    <div id="scoreboard">
        <div id="player">The legend: {playerScore.toString().padStart(2, '0')}</div>
        <div id="separator"> - </div>
        <div id="chameleon">{chameleonScore.toString().padStart(2, '0')} :Chameleon</div>
    </div>
    <div class="text">Hit the fly as many times as you can!</div>
</div>