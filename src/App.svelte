<script>
	import Quiz from './components/Quiz.svelte';

	let unique = {};
	function restart() {
		unique = {};
	}
</script>

<main>
	<!-- <h1>Hello {name}!</h1> -->
	<button id="restart" on:click={restart}>Restart</button>
	{#key unique}
		<Quiz {restart} />
	{/key}
</main>

<style>
	main {
		padding: 1rem 5rem;
	}
	#restart {
		float: right;
	}
</style>
