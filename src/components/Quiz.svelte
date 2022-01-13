<script>
	import Question from './Question.svelte';

	let numCorrect = 0;
	let numWrong = 0;
	let numUnanswered = 0;

	// const quizProm = getQuiz();
	let questionsProm = getQuiz();
	let questionIdx = 0;

	async function getQuiz() {
		const response = await fetch(
			'https://opentdb.com/api.php?amount=10&type=multiple'
		);

		const data = await response.json();
		console.log(data);
		const questionData = data.results.map(r => {
			return {
				questionHtml: r.question,
				answers: shuffle([r.correct_answer, ...r.incorrect_answers]),
				correctAnswer: r.correct_answer,
				answered: false,
				isCorrect: undefined,
				pickedAns: undefined,
				category: r.category,
			};
		});
		numUnanswered = questionsProm.length;
		return questionData;
	}

	function resetQuiz() {
		numCorrect = 0;
		numWrong = 0;
		numUnanswered = 0;
		questionIdx = 0;
		questionsProm = getQuiz();
	}

	function shuffle(unshuffled) {
		return unshuffled
			.map(value => ({ value, sort: Math.random() }))
			.sort((a, b) => a.sort - b.sort)
			.map(({ value }) => value);
	}
</script>

<div id="quiz-container">
	{#await questionsProm}
		<div>Loading Quiz...</div>
	{:then questions}
		<!-- <div on:click={resetQuiz}>reset</div> -->
		<h2 class="score">
			Score: {numCorrect}/{questions.length}
		</h2>

		<div class="question-wrapper">
			<Question
				questionHtml={questions[questionIdx].questionHtml}
				answers={questions[questionIdx].answers}
				correctAnswer={questions[questionIdx].correctAnswer}
				questionNum={questionIdx + 1}
				question={questions[questionIdx]}
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
		<div class="change-questions">
			{#if questionIdx !== 0}
				<button on:click={e => questionIdx--}>Previous Question</button>
			{/if}
			{#if questionIdx < questions.length - 1}
				{#if questions[questionIdx]?.pickedAns !== undefined}
					<button on:click={e => questionIdx++}>Next Question</button>
				{:else}
					<button
						disabled="true"
						on:click={e =>
							questionIdx < questions.length && questionIdx++}
						>Next Question</button
					>
				{/if}
			{/if}
		</div>
	{:catch error}
		<div>Error: {error}</div>
	{/await}
</div>

<style>
	#quiz-container {
		max-width: 600px;
		margin: auto;
	}
	.question-wrapper {
		margin: 3em 0;
	}
	.change-questions {
		display: flex;
		justify-content: flex-end;
	}
	.change-questions > button:last-child {
		margin-left: 1rem;
	}
	.change-questions > button {
		cursor: pointer;
	}
	.score {
		text-align: center;
		margin-bottom: 1rem;
	}
</style>
