<script>
	import { onMount, beforeUpdate, afterUpdate, onDestroy } from 'svelte';

	import { fade, blur, fly, slide } from 'svelte/transition';
	import Question from './Question.svelte';
	import Modal from './Modal.svelte';
	import { score } from '../store.js';

	export let restart;

	/* Constants */
	const numQuestions = 10;
	const correctToWin = 1;

	// let score = 0;
	// score.subscribe(val => (score = val));

	let numWrong = 0;
	let numUnanswered = 0;

	let questionsProm = getQuiz();
	let questionIdx = 0;
	let questionHeight;

	/* Modal stuff */
	// $: isComplete = numCorrect + numWrong === numQuestions;
	let isModalOpen = false;

	// reactive statement
	$: if ($score >= correctToWin) {
		isModalOpen = true;
	}

	onMount(() => {
		console.log('onMount');
	});
	beforeUpdate(() => {
		console.log('before update');
	});

	afterUpdate(() => {
		console.log('after update');
	});
	onDestroy(() => {
		console.log('onDestroy');
	});

	async function getQuiz() {
		console.log(`getting quiz`);
		const response = await fetch(
			`https://opentdb.com/api.php?amount=${numQuestions}&type=multiple`
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

	// instead of using unique to reset from parent, can also reassign the
	// getQuiz promise, which will re-await
	function resetQuiz() {
		score.set(0);
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

{#if isModalOpen}
	<Modal on:close={() => (isModalOpen = false)}>
		<h2>You won!</h2>
		<button on:click={restart}>Start Over</button>
	</Modal>
{/if}
<div id="quiz-container">
	{#await questionsProm}
		<div>Loading Quiz...</div>
	{:then questions}
		<!-- <div on:click={resetQuiz}>reset</div> -->
		<h2 class="score">
			Score: {$score}/{questions.length}
		</h2>
		{#key questionIdx}
			<div class="question-wrapper">
				<Question
					questionHtml={questions[questionIdx].questionHtml}
					answers={questions[questionIdx].answers}
					correctAnswer={questions[questionIdx].correctAnswer}
					questionNum={questionIdx + 1}
					question={questions[questionIdx]}
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
				<div class="change-questions">
					{#if questionIdx !== 0}
						<button on:click={e => questionIdx--}
							>Previous Question</button
						>
					{/if}
					{#if questionIdx < questions.length - 1}
						{#if questions[questionIdx]?.pickedAns !== undefined}
							<button on:click={e => questionIdx++}
								>Next Question</button
							>
						{:else}
							<button
								disabled="true"
								on:click={e =>
									questionIdx < questions.length &&
									questionIdx++}>Next Question</button
							>
						{/if}
					{/if}
				</div>
			</div>
		{/key}
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
		/* position: relative; */
		/* min-height: 100px; */
	}

	.change-questions {
		display: flex;
		justify-content: flex-end;
		/* position: relative; */
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
