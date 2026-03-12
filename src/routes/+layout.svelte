<script>
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<main>
{@render children()}
</main>

<style>
html, body {
  margin: 0;
  padding: 0;
  overflow: hidden; 
  width: 100%;
  height: 100%;
}

</style>