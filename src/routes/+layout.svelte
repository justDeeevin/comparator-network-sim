<script lang="ts">
  import './layout.css';
  import favicon from '$lib/assets/devin.jpg';

  let { children } = $props();
</script>

<svelte:head>
  <link rel="icon" href={favicon} />
  <title>comparator network simulator</title>
</svelte:head>
{@render children()}
