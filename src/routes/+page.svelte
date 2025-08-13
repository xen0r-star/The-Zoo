<script lang="ts">
    import ZooBanner from '$lib/Zoo.svelte';
    import Tree1 from '$lib/assets/Tree1.svg';
    import Tree2 from '$lib/assets/Tree2.svg';
    import Tree3 from '$lib/assets/Tree3.svg';
    import Tree4 from '$lib/assets/Tree4.svg';
</script>

<div style="height: 100vh; width: 100vw; position: absolute; z-index: 70;">
    <span class="material-symbols-outlined" style="font-size: 50px; position: absolute; bottom: 0; left: 50%; transform: translate(-50%, 0); color: white; margin-bottom: 40px; border: solid 1px; border-radius: 98989004px; font-variation-settings: 'wght' 100;">
        arrow_downward
    </span>
</div>

<div style="width: 100vw; background-color: #083537;">
    <h1 class="title" style="z-index: 50; position: sticky; top: 40%; transform: translateY(-50%); width: 100%;">Chameleon</h1>

    <div style="position: relative; height: 100vh; margin-top: 70vh;">
        <img src={Tree1} alt="Tree 1" style="position: absolute; height: 110%; left: 0; transform: translate(-47%, -7%); z-index:20" />
        <img src={Tree2} alt="Tree 2" style="position: absolute; height: 110%; right: 0; transform: translate(50%, -7%); z-index:30" />
        <img src={Tree3} alt="Tree 3" style="position: absolute; width: 100vw; left: 50%; transform: translate(-50%, -24%); z-index: 0" />
        <img src={Tree4} alt="Tree 4" style="position: absolute; width: 100vw; left: 50%; transform: translate(-50%, -26%); z-index: 60" />
    </div>
</div>

<div style="background-color: blueviolet; height: 100vh; position: relative; z-index:100;">
    <div style="height: 100px"></div>
    <h2 class="subtitle" style="z-index:50">A SvelteKit project</h2>
    <p class="description" style="z-index:50">This is a simple SvelteKit project with a custom banner and some SVG trees.</p>

</div>

<ZooBanner />