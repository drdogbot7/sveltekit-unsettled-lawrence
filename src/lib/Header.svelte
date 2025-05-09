<script>
	import { page } from '$app/stores';
	export let nav = [
		{
			title: 'Map',
			path: '/'
		},
		{
			title: 'About',
			path: '/about'
		},
		{
			title: 'Methodology',
			path: '/methodology'
		},
		{
			title: 'Analysis',
			path: '/analysis'
		},
		{
			title: 'Credits',
			path: '/credits'
		}
	];
</script>

<header class="z-10 z-10 flex flex-grow-0 items-center gap-6 bg-white p-2 shadow-sm">
	<h1 class="text-xl font-bold"><a href="/">Unsettled Lawrence</a></h1>
	<ul class="flex items-center gap-6">
		{#each nav as navItem}
			<li>
				<a
					on:click={() => closeNav()}
					class="block p-2 {$page.url.pathname === navItem.path ? 'font-bold' : 'hover:underline'}"
					href={navItem.path}>{navItem.title}</a
				>
			</li>
		{/each}
	</ul>
</header>
