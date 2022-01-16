<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();
</script>

<div class="modal-bg">
	<div class="modal">
		<button on:click={() => dispatch('close')}>Close</button>
		<slot />
	</div>
</div>

<style lang="scss">
	.modal-bg {
		position: fixed;
		top: 0;
		left: 0;
		bottom: 0;
		right: 0;
		background: rgba(0, 0, 0, 0.5);
		.modal {
			background-color: white;
		}
	}
</style>
