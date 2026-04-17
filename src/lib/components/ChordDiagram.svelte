<!-- TODO - How to remove small arcs? -->
<!-- TODO - How to make chart load/increase less abrupt? -->
<script>
	import { labels, matrix } from '$lib/data/deputados-adj-matrix.json';
	import { descending } from 'd3-array';
	import { chord, ribbon, chordDirected, ribbonArrow } from 'd3-chord';
	import { arc } from 'd3-shape';

	let { width, height } = $props();

	const defaultRibbonOpacity = 0.75;
	const defaultArcOpacity = 1;
	const nonHoveredOpacity = 0.3;
	const newMatrix = matrix.map((row, i) => row.map((value, j) => (i === j ? 0 : value)));
	const newGroup = 'a';

	const chordGenerator = chordDirected().padAngle(0.05).sortSubgroups(descending);
	const chords = chordGenerator(newMatrix);

	let outerRadius = $derived(Math.min(width, height) / 2 - 80);
	let innerRadius = $derived(outerRadius - 15);

	let arcGenerator = $derived(arc().innerRadius(innerRadius).outerRadius(outerRadius));
	let ribbonGenerator = $derived(ribbonArrow().radius(innerRadius - 5));

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
				? defaultArcOpacity
				: nonHoveredOpacity;
		}

		return group.index === hoveredGroupIndex ? defaultArcOpacity : nonHoveredOpacity;
	}

	function getRibbonOpacity(chord) {
		if (hoveredGroupIndex === null && hoveredRibbonKey === null) return defaultRibbonOpacity;

		if (hoveredRibbonKey !== null) {
			return `${chord.source.index}-${chord.target.index}` === hoveredRibbonKey
				? 1
				: nonHoveredOpacity;
		}

		return isRibbonConnectedToGroup(chord, hoveredGroupIndex)
			? defaultRibbonOpacity
			: nonHoveredOpacity;
	}

	function getLabelOpacity(group) {
		return getArcOpacity(group);
	}
</script>

<svg {width} {height}>
	<g transform="translate({width / 2}, {height / 2})">
		<!-- Arcs -->
		{#each chords.groups as group (group.index)}
			<path
				d={arcGenerator(group)}
				fill="steelblue"
				stroke="none"
				opacity={getArcOpacity(group)}
				class={labels[group.index]}
				onmouseenter={() => handleGroupEnter(group)}
				onmouseleave={clearHover}
				role="none"
			/>
		{/each}

		<!-- Ribbons -->
		<!-- Fix ribbon hover -->
		{#each chords as chord}
			<path
				d={ribbonGenerator(chord)}
				fill="steelblue"
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
				{labels[group.index]}
			</text>
		{/each}
	</g>
</svg>

<style>
	path {
		transition: opacity 500ms ease;
	}
</style>
