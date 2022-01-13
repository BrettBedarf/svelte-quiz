<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	export let questionHtml;
	export let answers = [];
	export let correctAnswer;
	export let questionNum;

	let isCorrect;
	let pickedAns;

	const pickAnswer = answer => {
		// do nothing if answer remains the same
		if (answer === pickedAns) return;
		if (answer === correctAnswer) {
			// if first time answering, add to correct results
			if (isCorrect === undefined) {
				dispatch('addToCorrect');
				dispatch('removeFromUnanswered');
			}
			// if prior answer was wrong, remove from wrong answers and add to correct
			if (isCorrect === false) {
				dispatch('removeFromWrong');
				dispatch('addToCorrect');
			}
		} else {
			// if first time answering, add to wrong
			if (isCorrect === undefined) {
				dispatch('addToWrong');
				dispatch('removeFromUnanswered');
			} else if (isCorrect === true) {
				// if prior answer was correct, remove from correct answers and add to
				// wrong
				dispatch('removeFromCorrect');
				dispatch('addToWrong');
			}
			// if prior answer was wrong, do nothing
		}
		pickedAns = answer;
		isCorrect = answer === correctAnswer;
	};
</script>

<div class="question-text">{`${questionNum}. `}{@html questionHtml}</div>

{#if isCorrect !== undefined}
	<h4 class="result" class:correct={pickedAns === correctAnswer}>
		{isCorrect ? 'Correct!' : 'Incorrect!'}
	</h4>
{:else}
	<h5>Pick an answer</h5>
{/if}
{#each answers as ans}
	<div class="answer-block">
		<button class="answer" on:click={e => pickAnswer(ans)} value={ans}
			>{@html ans}</button
		>
	</div>
{/each}

<style>
	.result.correct {
		color: green;
	}
	.result {
		color: red;
	}
	/* .answer {
		display: block;
	} */
</style>
