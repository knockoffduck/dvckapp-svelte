<script>
	import Shifts from '../components/Shifts.svelte';
	import { isOverlayOpen } from '../stores/OverlayStore';
	import '../styles/styles.css';
</script>

{#if $isOverlayOpen}
	<div class="fixed h-screen w-screen bg-opacity-50 bg-black flex justify-center items-center">
		<div class="modal-box flex flex-col items-center gap-6">
			<h3 class="font-bold text-2xl text-center">Successfully Added to Calendar</h3>
			<button
				on:click={() => {
					isOverlayOpen.set(false);
				}}
				class="btn w-1/4 font-bold">Close</button
			>
		</div>
	</div>
{/if}

<div>
	<Shifts />
</div>
