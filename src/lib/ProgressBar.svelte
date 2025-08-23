<script lang="ts">
    export let scrollProgress;
</script>

<style>
    .progress-container {
        width: 100vw; 
        height: 10px; 
        position: sticky; 
        bottom: 0; 
        z-index: 200;
    }

    .progress-bar {
        background: #00000096; 
        width: 100vw; 
        height: 10px; 
        bottom: 0; 
        z-index: 200;
    }

    .progress-bar > div {
        position: absolute;
        height: 10px; 
        background: #334517; 
        z-index: 210;
    }
</style>



<div class="progress-container">
    <div class="progress-bar">
        <div style="width: {scrollProgress}%;"></div>
    </div>
</div>