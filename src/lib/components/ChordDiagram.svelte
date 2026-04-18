<!-- TODO - How to remove small arcs? -->
<!-- TODO - How to make chart load/increase less abrupt? -->
<script>
	import { labels, matrix } from '$lib/data/deputados-adj-matrix.json';
	import { ascending, descending } from 'd3-array';
	import { chord, ribbon, chordDirected, ribbonArrow } from 'd3-chord';
	import { arc } from 'd3-shape';

	let { width, height } = $props();

	const defaultRibbonOpacity = 1;
	const defaultArcOpacity = 1;
	const hoveredOpacity = 1;
	const nonHoveredOpacity = 0.05;
	const newMatrix = matrix.map((row, i) => row.map((value, j) => (i === j ? 0 : value)));

	const activeIndices = labels
		.map((_, i) => i)
		.filter((i) => newMatrix[i].some((v) => v > 0) || newMatrix.some((row) => row[i] > 0));

	const partyAlignment = {
		PT: 'Esquerda',
		PCdoB: 'Esquerda',
		PSOL: 'Esquerda',
		PDT: 'Centro-esquerda',
		PSB: 'Centro-esquerda',
		CIDADANIA: 'Centro-esquerda',
		PV: 'Centro-esquerda',
		REDE: 'Centro-esquerda',
		MDB: 'Centro',
		PSD: 'Centro',
		PSDB: 'Centro',
		AVANTE: 'Centro',
		SOLIDARIEDADE: 'Centro',
		PP: 'Centro-direita',
		PODE: 'Centro-direita',
		UNIÃO: 'Centro-direita',
		PRD: 'Centro-direita',
		REPUBLICANOS: 'Direita',
		PL: 'Direita',
		NOVO: 'Direita',
		MISSÃO: 'Desconhecido'
	};

	const alignmentColor = {
		Esquerda: '#c0392b',
		'Centro-esquerda': '#e8a87c',
		Centro: '#aab7b8',
		'Centro-direita': '#5499c7',
		Direita: '#1a5276',
		Desconhecido: '#222'
	};

	const alignmentOrder = {
		Esquerda: 0,
		'Centro-esquerda': 1,
		Centro: 2,
		'Centro-direita': 3,
		Direita: 4,
		Desconhecido: 5
	};

	const legendGroups = [
		{ label: 'Esquerda', alignmentColor: '#c0392b' },
		{ label: 'Centro-esquerda', alignmentColor: '#e8a87c' },
		{ label: 'Centro', alignmentColor: '#aab7b8' },
		{ label: 'Centro-direita', alignmentColor: '#5499c7' },
		{ label: 'Direita', alignmentColor: '#1a5276' },
		{ label: 'Desconhecido', alignmentColor: '#222' }
	];

	const sortedIndices = activeIndices.sort((a, b) => {
		const orderA = alignmentOrder[partyAlignment[labels[a]] ?? 'Desconhecido'];
		const orderB = alignmentOrder[partyAlignment[labels[b]] ?? 'Desconhecido'];
		return orderA - orderB;
	});

	const filteredLabels = activeIndices.map((i) => labels[i]);
	const filteredMatrix = activeIndices.map((i) => activeIndices.map((j) => newMatrix[i][j]));

	function getColor(index) {
		return alignmentColor[getAlignment(index)];
	}

	function getAlignment(index) {
		return partyAlignment[filteredLabels[index]];
	}

	const chordGenerator = chordDirected()
		.padAngle(0.05)
		// .sortGroups(ascending)
		.sortSubgroups(ascending)
		.sortChords(ascending);
	const chords = chordGenerator(filteredMatrix);

	let outerRadius = $derived(Math.min(width, height) / 2 - 100);
	let innerRadius = $derived(outerRadius - 15);

	let arcGenerator = $derived(arc().innerRadius(innerRadius).outerRadius(outerRadius));
	let ribbonGenerator = $derived(
		ribbonArrow()
			.radius(innerRadius - 5)
			.headRadius(10)
	);

	let hoveredGroupIndex = $state(null);
	let hoveredRibbonKey = $state(null);
	let hoveredRibbonIndices = $state(null);

	function clearHover() {
		hoveredGroupIndex = null;
		hoveredRibbonKey = null;
		hoveredRibbonIndices = null;
	}

	function handleGroupEnter(group) {
		hoveredGroupIndex = group.index;
		hoveredRibbonKey = null;
		hoveredRibbonIndices = null;
	}

	function handleRibbonEnter(chord) {
		hoveredGroupIndex = null;
		hoveredRibbonKey = `${chord.source.index}-${chord.target.index}`;
		hoveredRibbonIndices = { source: chord.source.index, target: chord.target.index };
	}

	function isRibbonConnectedToGroup(chord, groupIndex) {
		return chord.source.index === groupIndex || chord.target.index === groupIndex;
	}

	function getArcOpacity(group) {
		if (hoveredGroupIndex === null && hoveredRibbonKey === null) return defaultArcOpacity;

		if (hoveredRibbonIndices !== null) {
			return hoveredRibbonIndices.source === group.index ||
				hoveredRibbonIndices.target === group.index
				? hoveredOpacity
				: nonHoveredOpacity;
		}

		return group.index === hoveredGroupIndex ? hoveredOpacity : nonHoveredOpacity;
	}

	function getRibbonOpacity(chord) {
		if (hoveredGroupIndex === null && hoveredRibbonKey === null) return defaultRibbonOpacity;

		if (hoveredRibbonKey !== null) {
			return `${chord.source.index}-${chord.target.index}` === hoveredRibbonKey
				? hoveredOpacity
				: nonHoveredOpacity;
		}

		return isRibbonConnectedToGroup(chord, hoveredGroupIndex) ? hoveredOpacity : nonHoveredOpacity;
	}

	function getLabelOpacity(group) {
		return getArcOpacity(group);
	}
</script>

<div class="wrapper">
	<svg {width} {height}>
		<g transform="translate({width / 2}, {height / 2})">
			<!-- Arcs -->
			{#each chords.groups as group (group.index)}
				<path
					d={arcGenerator(group)}
					fill={getColor(group.index)}
					stroke="none"
					opacity={getArcOpacity(group)}
					class={filteredLabels[group.index]}
					onmouseenter={() => handleGroupEnter(group)}
					onmouseleave={clearHover}
					role="none"
				/>
			{/each}

			<!-- Ribbons -->
			{#each chords as chord}
				<path
					d={ribbonGenerator(chord)}
					fill={getColor(chord.target.index)}
					stroke="none"
					opacity={getRibbonOpacity(chord)}
					onmouseenter={() => {
						handleRibbonEnter(chord);
					}}
					onmouseleave={clearHover}
					role="none"
				/>
			{/each}

			<!-- Labels -->
			{#each chords.groups as group}
				{@const angle = (group.startAngle + group.endAngle) / 2}
				{@const rotate = angle * (180 / Math.PI) - 90}
				<text
					transform="rotate({rotate}) translate({outerRadius + 12}) {angle > Math.PI
						? 'rotate(180)'
						: ''}"
					text-anchor={angle > Math.PI ? 'end' : 'start'}
					dominant-baseline="central"
					font-size="11"
				>
					{filteredLabels[group.index]}
				</text>
			{/each}
		</g>
	</svg>
	<div class="legend">
		{#each legendGroups as legend}
			<div class="legend-item">
				<span class="swatch" style="background: {legend.alignmentColor}"></span>
				<span>{legend.label}</span>
			</div>
		{/each}
	</div>
</div>

<style>
	.wrapper {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	path {
		transition: opacity 500ms ease;
	}

	.legend {
		display: grid;
		align-items: center;
		justify-content: center;
		grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
		gap: 24px;
		width: 100%;
	}

	.legend-item {
		display: flex;
		align-items: center;
		gap: 6px;
		font-size: 11px;
	}

	.swatch {
		width: 10px;
		height: 10px;
		border-radius: 2px;
		flex-shrink: 0;
	}
</style>
