<script>
	import '../app.css';

	let { children } = $props();
</script>

<div class="flex h-screen flex-col justify-stretch">
	<header class="z-10 flex flex-grow-0 items-center gap-6 bg-white p-2 shadow-sm">
		<h1 class="text-xl font-bold"><a href="/">Unsettled Lawrence</a></h1>
		<ul class="flex items-center gap-6">
			<li><a href="/"><b>Map</b></a></li>
			<li><a class="hover:underline" href="about">About</a></li>
			<li><a class="hover:underline" href="methodology">Methodology</a></li>
			<li><a class="hover:underline" href="analysis">Analysis</a></li>
		</ul>
	</header>

	{@render children()}
</div>
