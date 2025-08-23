<script lang="ts">
    import TongueChameleon from '$lib/assets/HomePage/TongueChameleon.svg'
</script>

<style>
    .section-container {
        background-color: var(--primary); 
        position: relative; 
        z-index:100; 
        display: flex; 
        flex-direction: row; 
        align-items: center; 
        justify-content: space-evenly;
    }

    #ChameleonTongue {
        position: absolute; 
        top: 0; 
        left: 0; 
        transform: rotate(12.36deg) translate(-7%, 5%); 
        filter: drop-shadow(-5px 5px 0px rgba(0, 0, 0, 0.25)); 
        z-index: 110;
    }

    #ChameleonTongueText {
        background: var(--white-gradient); 
        color: var(--black-text); 
        margin: 150px 0 75px 0; 
        max-width: 1000px; 
        border-radius: 18px; 
        box-shadow: 0 4px 24px #0002; 
        padding: 48px 32px; 
        font-family: 'Segoe UI', 'Arial', sans-serif; 
        position: relative; 
        display: flex; 
        flex-direction: column;
    }

    #ChameleonTongueText h2 {
        font-size: 2em; 
        font-weight: bold; 
        margin-bottom: 18px; 
        letter-spacing: 0.02em; 
        text-align: left;
    }

    #ChameleonTongueText p {
        font-size: 1.18em; 
        line-height: 1.6;
    }
</style>



<div class="section-container">
    <div id="ChameleonTongue">
        <img src={TongueChameleon} alt="Chameleon with tongue out" />
    </div>

    <div id="ChameleonTongueText">
        <h2>
            The chameleon's tongue
        </h2>
        <p>
            The chameleon has an amazing weapon: its tongue.<br>
            It can measure up to twice the length of its body and shoots out in less than a tenth of a second.
            Propelled like a spring, it reaches its prey with incredible precision.<br>
            Its sticky tip works like a suction cup that closes around the insect and prevents it from escaping.
            Thanks to this technique, the chameleon catches its meals without having to move from its branch.
        </p>
    </div>
</div>