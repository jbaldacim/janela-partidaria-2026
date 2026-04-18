<script>
	import ChordDiagram from '$lib/components/ChordDiagram.svelte';
	let minWidth = $state(400);
	let windowWidth = $state(0);

	let width = $derived(Math.max(minWidth, windowWidth));
	const height = 800;
</script>

<div bind:clientWidth={windowWidth}>
	<h1>Janela partidária 2026</h1>
	<h2>Fluxo de deputados entre partidos</h2>
	<p>Partidos alinhados por localização no espectro político</p>
	<ChordDiagram {width} {height} />
</div>

<style>
	div {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	h1 {
		text-transform: uppercase;
		letter-spacing: 0.1rem;
	}
</style>
