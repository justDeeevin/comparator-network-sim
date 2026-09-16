<script lang="ts">
  import { type HTMLInputAttributes } from 'svelte/elements';

  let {
    value = $bindable(),
    min,
    max,
    ...props
  }: { min?: number; max?: number } & HTMLInputAttributes = $props();
</script>

<input
  class="field-sizing-content min-w-3 border border-black px-1 text-center"
  bind:value
  oninput={() => {
    value = parseInt(value.toString().replace(/[^0-9]/g, '')) || 0;
    if (min !== undefined && value < min) value = min;
    if (max !== undefined && value > max) value = max;
  }}
  {...props}
/>
