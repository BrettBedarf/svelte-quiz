<script>
	import Question from './Question.svelte';

	let numCorrect = 0;
	let numWrong = 0;
	let numUnanswered = 0;

	const quizProm = getQuiz();
	let questions = [];

	async function getQuiz() {
		const response = await fetch(
			'https://opentdb.com/api.php?amount=10&type=multiple'
		);

		const data = await response.json();
		console.log(data);
		questions = data.results;
		numUnanswered = questions.length;
		return data;
	}

	function shuffle(unshuffled) {
		return unshuffled
			.map(value => ({ value, sort: Math.random() }))
			.sort((a, b) => a.sort - b.sort)
			.map(({ value }) => value);
	}
</script>

<div>
	{#await quizProm}
		<div>Loading Quiz...</div>
	{:then quiz}
		<div>
			Correct:{numCorrect}
		</div>
		<div>
			Wrong:{numWrong}
		</div>
		<div>
			Unanswered:{numUnanswered}
		</div>
		{#each quiz.results as question, i}
			<div class="question-wrapper">
				<Question
					questionHtml={question.question}
					answers={shuffle([
						...question.incorrect_answers,
						question.correct_answer,
					])}
					correctAnswer={question.correct_answer}
					questionNum={i + 1}
					on:addToCorrect={() => {
						numCorrect++;
					}}
					on:removeFromCorrect={() => {
						numCorrect--;
					}}
					on:addToWrong={() => {
						numWrong++;
					}}
					on:removeFromWrong={() => {
						numWrong--;
					}}
					on:removeFromUnanswered={() => {
						numUnanswered--;
					}}
				/>
			</div>
		{/each}
		<!-- <pre class="question-text">{@html questions[0].question}</pre>
		{#if result}
			<h4 class="result" class:correct={pickedAns === correctAns}>
				{result}
			</h4>
		{:else}
			<h5>Pick an answer</h5>
		{/if}
		{#each shuffle( [...questions[0].incorrect_answers, questions[0].correct_answer] ) as ans}
			<div class="answer-block">
				<button
					class="answer"
					on:click={e => pickAnswer(ans, questions[0].correct_answer)}
					value={ans}>{ans}</button
				>
			</div>
		{/each} -->
	{:catch error}
		<div>Error: {error}</div>
	{/await}
</div>

<style>
	.question-wrapper {
		margin: 3em;
	}
</style>
