<script>
	import ChordDiagram from '$lib/components/ChordDiagram.svelte';
	let minWidth = $state(400);
	let windowWidth = $state(0);

	let width = $derived(Math.max(minWidth, windowWidth));
	const height = 800;
</script>

<div bind:clientWidth={windowWidth}>
	{#if windowWidth !== 0}
		<div class="loaded">
			<h1>Janela partidária 2026</h1>
			<h2>Fluxo de deputados entre partidos</h2>
			<p>Partidos organizados por alinhamento no espectro político</p>
			<ChordDiagram {width} {height} />
		</div>
	{/if}
</div>

<style>
	div {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.loaded {
		opacity: 0;
		animation: fadeIn 500ms ease forwards;
	}

	@keyframes fadeIn {
		to {
			opacity: 1;
		}
	}

	h1 {
		text-transform: uppercase;
		letter-spacing: 0.1rem;
	}
</style>
