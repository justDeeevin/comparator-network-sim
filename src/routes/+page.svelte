<script lang="ts">
  import NumAdjust from '$lib/components/num-adjust.svelte';
  import Button from '$lib/components/button.svelte';
  import NumInput from '$lib/components/num-input.svelte';
  import { SvelteMap as Map } from 'svelte/reactivity';

  const initial_depth = 3;
  const initial_inputs = 4;

  let depth = $state(initial_depth);
  let inputs = $state(Array.from({ length: initial_inputs }, (_, i) => i + 1));
  $effect(() => {
    for (let i = 0; i < inputs.length; i++)
      if (inputs[i] === undefined) inputs[i] = i + 1;
  });
  let source: [number, number] | undefined = $state();

  /**
    depth->row->connected row
  */
  let connections: Map<number, Map<number, number>> = $state(new Map());
  let crossings: Map<number, Set<number>> = $derived(
    new Map(
      connections
        .entries()
        .map(([k, connections]) => [
          k,
          new Set(
            connections
              .entries()
              .flatMap(([k, v]) =>
                Array.from({ length: v - k - 1 }, (_, i) => k + i + 1),
              ),
          ),
        ]),
    ),
  );

  let outputs = $derived.by(() => {
    if (connections.size === 0) return inputs;

    let output = Array.from(inputs);
    for (let d = 0; d < depth; d++) {
      let nodes = connections.get(d);
      if (nodes === undefined) continue;
      for (let i = 0; i < output.length; i++) {
        let connection = nodes.get(i);
        if (connection !== undefined && output[connection] < output[i])
          [output[i], output[connection]] = [output[connection], output[i]];
      }
    }
    return output;
  });

  function connect(depth: number, source: number, target: number) {
    if (source > target) [source, target] = [target, source];
    let nodes = connections.getOrInsertComputed(depth, () => new Map());

    // If either node is already targeted, break its connection
    for (const n of [source, target]) {
      let deleteme_source = nodes.entries().find(([_, d]) => d === n)?.[0];
      if (deleteme_source !== undefined) nodes.delete(deleteme_source);
    }

    nodes.delete(target);
    if (source !== target) nodes.set(source, target);
  }

  function shuffled<T>(array: T[]): T[] {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }

    return array;
  }
</script>

<div class="grid h-screen w-screen grid-rows-3">
  <div
    class="
      row-2
      flex
      flex-col
      items-center
      justify-center
      gap-7
    "
  >
    {#each inputs, row}
      <div class="flex items-center gap-3">
        <NumInput bind:value={inputs[row]} />
        <div class="flex h-1 items-center gap-10 bg-black">
          {#each { length: depth }, depth}
            <button
              title="Node"
              class={`
                flex
                size-3.5
                cursor-pointer
                justify-center
                rounded-full
                border
                border-black
                ${source?.[0] === depth && source?.[1] === row ? 'bg-red-500' : 'bg-gray-200'}
              `}
              onclick={() => {
                if (source === undefined) source = [depth, row];
                else {
                  if (depth == source[0]) connect(source[0], source[1], row);
                  source = undefined;
                }
              }}
            >
              {#if connections.get(depth)?.has(row) || crossings
                  .get(depth)
                  ?.has(row)}
                <div
                  class={`
                    h-14
                    w-1
                    bg-black
                    ${crossings.get(depth)?.has(row) ? '' : ' -z-1'}
                  `}
                ></div>
              {/if}
            </button>
          {/each}
        </div>
        <p>{outputs[row]}</p>
      </div>
    {/each}
  </div>
  <div class="-row-1 flex justify-center gap-3">
    <NumAdjust label="Depth" bind:value={depth} min={1} />
    <NumAdjust label="Inputs" bind:value={inputs.length} min={2} />
    <Button onclick={() => (inputs = shuffled(inputs))}>Shuffle inputs</Button>
    <Button onclick={() => (connections = new Map())}>Clear connections</Button>
  </div>
</div>
