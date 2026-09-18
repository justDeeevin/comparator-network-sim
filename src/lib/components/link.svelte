<script lang="ts">
  import { type HTMLAnchorAttributes } from 'svelte/elements';
  const {
    class: className,
    children,
    ...props
  }: HTMLAnchorAttributes = $props();
</script>

<a class={'text-blue-500 underline ' + className} {...props}>
  {@render children?.()}
</a>
