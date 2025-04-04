<script lang="ts">
	// import '../app.css';
	import './jogo_da_velha.css';

	let { children } = $props();
</script>

<svelte:head>
	<title>Marcando o jogo</title>
</svelte:head>

{@render children()}
