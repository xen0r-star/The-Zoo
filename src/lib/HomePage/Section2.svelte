<script lang="ts">
    import UpsideDownChameleon from '$lib/assets/HomePage/UpsideDownChameleon.svg';
</script>

<style>
    .section-container {
        background-color: var(--secondary); 
        position: relative; 
        z-index:100; 
        display: flex; 
        flex-direction: row; 
        align-items: center; 
        justify-content: space-evenly;
    }

    #ChameleonUpsideDown {
        position: absolute; 
        top: 0; 
        right: 0; 
        transform: rotate(12.36deg) translate(11%, 15%); 
        filter: drop-shadow(-5px 5px 0px rgba(0, 0, 0, 0.25)); 
        z-index: 110;
    }

    .text-container {
        background: var(--red-gradient); 
        gap: 50px; 
        margin: 100px 0; 
        max-width: 1000px; 
        border-radius: 18px; 
        box-shadow: 0 4px 24px #0002; 
        padding: 48px 32px; 
        color: #234a1a; 
        font-family: 'Segoe UI', 'Arial', sans-serif; 
        position: relative; 
        text-align: center; 
        display: flex; 
        flex-direction: row;
    }

    #FascinatingCamouflageText {
        flex: 2; 
        max-width: 60%; 
        position: relative;
    }

    #FascinatingCamouflageText h2 {
        color: var(--white-text); 
        font-size: 2em; 
        font-weight: bold; 
        margin-bottom: 18px; 
        letter-spacing: 0.02em; 
        text-align: left;
    }

    #FascinatingCamouflageText p {
        font-size: 1.18em; 
        line-height: 1.6; 
        margin: 0; 
        text-align: left; 
        color: var(--white-text);
    }
</style>



<div class="section-container">
    <div id="ChameleonUpsideDown">
        <img src={UpsideDownChameleon} alt="Upside Down Chameleon" />
    </div>
    
    <div class="text-container">
        <div id="FascinatingCamouflageText">
            <h2>
                Fascinating camouflage
            </h2>
            <p>
                Chameleons are famous for their ability to change color.<br>
                This transformation not only helps them blend into their surroundings, but also allows them to communicate with other chameleons and regulate their body temperature.
                Patterns and shades vary depending on the species, mood, light, and heat.
                Some species display bright colors to intimidate rivals or attract mates, while others adopt more subtle tones to remain invisible to predators.
            </p>
        </div>
    </div>
</div>
