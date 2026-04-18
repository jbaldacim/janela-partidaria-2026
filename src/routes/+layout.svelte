<script lang="ts">
	import favicon from '$lib/assets/favicon.svg';
	import '@fontsource-variable/outfit/wght.css';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<div>
	{@render children()}
</div>

<style>
	:global(*, *::before, *::after) {
		box-sizing: border-box;
		margin: 0;
	}
	:global(body) {
		font-family: 'Outfit Variable', sans-serif;
		background: #ddd;
		max-height: 100vh;
	}

	div {
		padding: 1rem;
	}
</style>
