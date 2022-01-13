<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	export let questionHtml;
	export let answers = [];
	export let correctAnswer;
	export let questionNum;
	export let question;

	const pickAnswer = answer => {
		// do nothing if answer remains the same
		if (answer === question.pickedAns) return;
		if (answer === correctAnswer) {
			// if first time answering, add to correct results
			if (question.isCorrect === undefined) {
				dispatch('addToCorrect');
				dispatch('removeFromUnanswered');
			}
			// if prior answer was wrong, remove from wrong answers and add to correct
			if (question.isCorrect === false) {
				dispatch('removeFromWrong');
				dispatch('addToCorrect');
			}
		} else {
			// if first time answering, add to wrong
			if (question.isCorrect === undefined) {
				dispatch('addToWrong');
				dispatch('removeFromUnanswered');
			} else if (question.isCorrect === true) {
				// if prior answer was correct, remove from correct answers and add to
				// wrong
				dispatch('removeFromCorrect');
				dispatch('addToWrong');
			}
			// if prior answer was wrong, do nothing
		}
		question.pickedAns = answer;
		question.isCorrect = answer === correctAnswer;
		question.answers = question.answers;
	};

	function isCorrect(ans) {
		if (ans === question.correctAnswer) {
			return true;
		} else {
			return false;
		}
	}
</script>

<div class="question-header">
	<span class="question-num">Question {questionNum}</span>
	<span class="question-category">({question.category})</span>
</div>
<div class="question-text">{@html questionHtml}</div>

{#if question.isCorrect !== undefined}
	<h4 class="result" class:correct={question.pickedAns === correctAnswer}>
		{question.isCorrect ? 'Correct!' : 'Incorrect!'}
	</h4>
{:else}
	<h5>Pick an answer</h5>
{/if}
{#each answers as ans}
	<div
		class="answer"
		class:is-answered={question.pickedAns !== undefined}
		class:correct={question.pickedAns && isCorrect(ans)}
		class:wrong={question.pickedAns &&
			!isCorrect(ans) &&
			question.pickedAns === ans}
		on:click={!question.pickedAns && (e => pickAnswer(ans))}
		value={ans}
	>
		{@html ans}
	</div>
{/each}

<style>
	.answer:not(.is-answered):hover {
		background-color: rgb(236, 236, 236);
	}

	.answer {
		cursor: pointer;
		padding: 0.5rem;
		border: 1px solid black;
		margin: 1rem 0;
		border-radius: 10px;
	}

	.result.correct {
		color: rgb(99, 206, 99);
	}
	.result {
		color: rgb(245, 105, 105);
	}
	.answer.correct {
		background-color: rgb(99, 206, 99);
		color: white;
	}
	.answer.wrong {
		background-color: rgb(245, 105, 105);
		color: white;
	}
	.question-text {
		min-height: 2rem;
	}
	.question-header {
		margin-bottom: 2rem;
	}
	.question-num {
		font-size: 1.25rem;
		font-weight: bold;
		margin-bottom: 1rem;
	}
	.question-category {
		font-size: 0.85rem;
		font-style: italic;
		padding-left: 0.5rem;
	}
</style>
