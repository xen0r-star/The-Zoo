<script lang="ts">
    import DangerGraph from '$lib/assets/HomePage/DangerGraph.svg'
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

    #ChameleonDangerText {
        background: var(--blue-gradient); 
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
        flex-direction: row-reverse;
    }

    #graphDanger {
        width: 300px; 
        height: 300px; 
        display: flex; 
        flex: 1; 
        align-items: center; 
        justify-content: center; 
        position: relative;
    }

    .section-text {
        flex: 2; 
        max-width: 60%; 
        position: relative; 
        color: var(--white-text);
    }

    .section-text h2 {
        font-size: 2em; 
        font-weight: bold; 
        margin-bottom: 18px; 
        letter-spacing: 0.02em; 
        text-align: left;
    }

    .section-text p {
        font-size: 1.18em; 
        line-height: 1.6; 
        margin: 0; 
        text-align: left;
    }

    .section-text a {
        position: absolute; 
        bottom: 0; 
        width: 100%; 
        right: 0;
    }

    .strong-text {
        color: var(--important-text); 
        font-weight: bold; 
        font-size: 1.2em;
    }

    .link-text {
        text-align: left; 
        color: var(--tertiary-text);
    }
</style>



<div class="section-container">
    <div id="ChameleonDangerText">
        <div id="graphDanger">
            <img src={DangerGraph} alt="Graph showing the danger level of chameleons" class="full" />
        </div>
        <div class="section-text">
            <h2>
                Chameleons in danger!
            </h2>
            <p>
                Nearly <strong class="strong-text">38%</strong> of chameleons are at risk of extinction. The destruction of their forests and global warming are making their survival increasingly difficult.<br>
                Protecting nature also means saving these incredible lizards.
            </p>
            <a id="UICNLink" href="https://iucn.org/" target="_blank">
                <p class="link-text">International Union for Conservation of Nature (UICN)</p>
            </a>
        </div>
    </div>
</div>