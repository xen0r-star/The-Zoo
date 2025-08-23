<script lang="ts">
    export let unlockLevel: number;
    export let onClick: () => void;
</script>

<style>
    .section-container {
        background-color: var(--secondary); 
        height: 300px; 
        position: relative; 
        z-index:100; 
        display: flex; 
        flex-direction: column; 
        align-items: center; 
        justify-content: center; 
        flex-wrap: nowrap;
    }

    h3 {
        font-size: 1.25em; 
        margin-bottom: 25px; 
        color: var(--black-text); 
        padding: 25px; 
        text-align: center;
    }

    .big-button {
        color: #fff; 
        font-size: 1.35em; 
        font-weight: bold; 
        padding: 18px 48px; 
        border: none; 
        border-radius: 16px;
        cursor: pointer; 
        letter-spacing: 0.04em;
        transition: all 0.15s ease-in-out; 
        margin: 0 auto; 
        display: flex; 
        flex-direction: row; 
        align-items: center; 
        gap: 10px;
    }

    .big-button:hover {
        transform: translate(7px, 7px);
        box-shadow: rgb(59, 59, 59) 0px 0px 0px !important;
    }

    .big-button > span {
        font-weight: bold;
    }

    p {
        margin: 20px; 
        text-align: center; 
        color: var(--secondary-text);
    }
</style>



<div class="section-container">
    <h3>Are you ready to challenge me and prove your skills?</h3>
    <button onclick={onClick} class="big-button" style="background: {unlockLevel >= 100 ? '#90ad28' : '#8e8e8e'}; box-shadow: 7px 7px 0px {unlockLevel >= 100 ? '#627814' : '#3b3b3b'};">
        <span class="material-symbols-outlined bannerIcon">{unlockLevel >= 100 ? 'lock_open_right' : 'lock'}</span> Meet Chameleon
    </button>
    <p>Read the site content to unlock it. ({unlockLevel}%)</p>
</div>