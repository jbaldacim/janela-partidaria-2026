<script>
	import ChordDiagram from '$lib/components/ChordDiagram.svelte';
	let minWidth = $state(400);
	let windowWidth = $state(0);

	let width = $derived(Math.max(minWidth, windowWidth));
	const height = 800;
</script>

<div bind:clientWidth={windowWidth}><ChordDiagram {width} {height} /></div>

<style>
	div {
		display: flex;
		align-items: center;
		justify-content: center;
		font-family: Helvetica, sans-serif;
	}
</style>
