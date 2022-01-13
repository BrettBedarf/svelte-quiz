<script>
	export let name;
	import Quiz from './components/Quiz.svelte';
</script>

<main>
	<h1>Hello {name}!</h1>
	<Quiz quizName="Sick Quiz" />
</main>

<style>
</style>
