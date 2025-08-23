<script lang="ts">
    import IdCard from '$lib/assets/IdCard.svg';
</script>

<style>
    .section-container {
        background-color: var(--primary); 
        position: relative; 
        z-index: 100; 
        display: flex; 
        flex-direction: row; 
        align-items: center; 
        justify-content: space-evenly;
    }

    .id-card-content {
        max-width: 1000px; 
        display: flex; 
        flex-direction: column; 
        width: 100%; 
        margin: 50px auto; 
        position: relative; 
        background: var(--tertiary); 
        border-radius: 15px; 
        box-shadow: 0px 4px 10px rgba(0,0,0,0.1);
    }
    
    .id-card-content h2 {
        display: flex; 
        align-items: center; 
        justify-content: center; 
        border-radius: 15px 15px 0 0; 
        width: 100%; 
        height: 42px; 
        background: #002564; 
        text-align: center; 
        color: var(--white-text); 
        font-weight: bold; 
        font-size: 1.15em;
    }

    #IdCard {
        color: var(--black-text); 
        display: flex; 
        flex-direction: row; 
        justify-content: space-evenly; 
        padding: 32px; 
        align-items: center;
    }

    .picture-id-card {
        width: 30%; 
        height: 100%; 
        display: flex; 
        flex-direction: column; 
        justify-content: center; 
        align-items: center;
    }

    .picture-id-card img {
        min-width: 100px; 
        background: #cafffb; 
        width: 80%; 
        border-radius: 20px; 
        box-shadow: 0px 4px 10px rgba(0,0,0,0.1);
    }

    .picture-id-card p:first-of-type {
        margin-top: 10px; 
        font-weight: bold; 
        font-size: 1.25em;
    }

    .picture-id-card p:nth-last-of-type(2) {
        font-size: 1em;
    }

    #IdCardText {
        width: 55%; 
        height: 100%; 
        display: flex; 
        flex-direction: column; 
        justify-content: center;
    }

    .element-content {
        display: flex; 
        align-items: flex-start; 
        flex-direction: row; 
        margin-block: 5px;
    }

    .element-content p {
        font-size: 1.1em; 
        display: flex; 
        align-items: flex-start;
    }
    
    .icon-id-card {
        font-size: 1.5em; 
        margin-right: 10px;
    }

    #IdCardBarCode {
        font-family: 'Libre Barcode 39'; 
        font-size: 200%; 
        text-align: right; 
        height: 50px;
    }
</style>



<div class="section-container">
    <div class="border"></div>

    <div class="id-card-content">
        <h2>ID CARD</h2>
        <div id="IdCard">
            <div class="picture-id-card">
                <img src={IdCard} alt="Chameleon" />
                <p>Chameleon</p>
                <p>Chamaeleonidae</p>
            </div>
            <div id="IdCardText">
                <span class="element-content">
                    <p>
                        <span class="material-symbols-outlined icon-id-card">pets</span>
                        <span><strong>Class</strong>: Reptilia</span>
                    </p>
                </span>
    
                <span class="element-content">
                    <p>
                        <span class="material-symbols-outlined icon-id-card">straighten</span>
                        <span><strong>Size</strong>: From 3 cm up to 70 cm</span>
                    </p>
                </span>
    
                <span class="element-content">
                    <p>
                        <span class="material-symbols-outlined icon-id-card">balance</span>
                        <span><strong>Weight</strong>: 15g to 200g depending on the species</span>
                    </p>
                </span>
    
                <span class="element-content">
                    <p>
                        <span class="material-symbols-outlined icon-id-card">hourglass_empty</span>
                        <span><strong>Lifespan</strong>: 5 to 10 years on average</span>
                    </p>
                </span>
    
                <span class="element-content">
                    <p>
                        <span class="material-symbols-outlined icon-id-card">park</span>
                        <span><strong>Habitat</strong>: Tropical forests, savannas, mountains, and even semi-desert areas</span>
                    </p>
                </span>
    
                <span class="element-content">
                    <p>
                        <span class="material-symbols-outlined icon-id-card">public</span>
                        <span><strong>Distribution</strong>: Africa, Madagascar, southern Europe, ...</span>
                    </p>
                </span>
    
                <span class="element-content">
                    <p>
                        <span class="material-symbols-outlined icon-id-card">restaurant</span>
                        <span><strong>Diet</strong>: Mostly insects (crickets, flies, grasshoppers)</span>
                    </p>
                </span>
    
                <div id="IdCardBarCode">bit.ly/47xw5Zh</div>
            </div>
        </div>
    </div>
</div>