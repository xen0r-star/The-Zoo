<script lang="ts">
	import '../app.css';
	import '$lib/style/media.css';
	import '$lib/style/keyframes.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<link href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@48,400,0,0" rel="stylesheet" />
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com">
	<link href="https://fonts.googleapis.com/css2?family=Libre+Barcode+39&display=swap" rel="stylesheet">

	<link rel="preload" href="/facts.json" as="fetch" type="application/json" crossorigin="anonymous">
	<link rel="preload" href="/skyColor.json" as="fetch" type="application/json" crossorigin="anonymous">
	<link rel="preload" href="/chameleonPalettes.json" as="fetch" type="application/json" crossorigin="anonymous">

	<title>The Zoo - Chameleon</title>
</svelte:head>

{@render children?.()}
