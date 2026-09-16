<script lang="ts">
  import type { HTMLButtonAttributes } from 'svelte/elements';

  let { children, class: className, ...props }: HTMLButtonAttributes = $props();
</script>

<button
  class={`
    cursor-pointer
    rounded-sm
    border
border-black
    bg-gray-200
    px-1
    ${className}
  `}
  {...props}
>
  {@render children?.()}
</button>
