<script lang="ts">
	import { labels, matrix } from '$lib/data/deputados-adj-matrix.json';
	import { chord } from 'd3-chord';
	import { descending } from 'd3-array';
	let { width, height } = $props();

	const chordGenerator = chord().padAngle(0.05).sortSubgroups(descending);
	const chords = chordGenerator(matrix);
</script>

<svg {width} {height} fill="firebrick"></svg>
