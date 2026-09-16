<script lang="ts">
  import Button from './button.svelte';
  import NumInput from './num-input.svelte';

  let {
    value = $bindable(),
    label,
    min,
    max,
  }: {
    value: number;
    label?: string;
    min?: number;
    max?: number;
  } = $props();
</script>

<div class="flex gap-0.5">
  <Button
    class="size-6"
    onclick={() => {
      if (min === undefined || value > min) value--;
    }}
  >
    -
  </Button>
  {label}: <NumInput bind:value {min} {max} />
  <Button
    class="size-6"
    onclick={() => {
      if (max === undefined || value < max) value++;
    }}
  >
    +
  </Button>
</div>
