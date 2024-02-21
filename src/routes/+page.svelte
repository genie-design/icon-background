<script lang="ts">
	$: count = 0;
</script>

<button on:click={() => count++}>
	clicks: {count}
</button>
