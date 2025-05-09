<script>
	import '../app.css';
	import Header from '$lib/Header.svelte';

	let { children } = $props();
</script>

<svelte:head>
	<title>Unsettled Lawrence</title>
</svelte:head>

<div class="flex h-screen flex-col justify-stretch">
	<Header />
	{@render children()}
</div>
