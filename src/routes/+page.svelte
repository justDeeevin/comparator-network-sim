<script lang="ts">
  import NumAdjust from '$lib/components/num-adjust.svelte';
  import Button from '$lib/components/button.svelte';
  import NumInput from '$lib/components/num-input.svelte';
  import Link from '$lib/components/link.svelte';
  import { SvelteMap as Map } from 'svelte/reactivity';

  const initial_depth = 3;
  const initial_inputs = 4;

  let depth = $state(initial_depth);
  const inputs = $state(
    Array.from({ length: initial_inputs }, (_, i) => i + 1),
  );
  $effect(() => {
    for (let i = 0; i < inputs.length; i++)
      if (inputs[i] === undefined) inputs[i] = i + 1;
  });
  let source: [number, number] | undefined = $state();

  /**
    depth->row->connected row
  */
  const connections: Map<number, Map<number, number>> = $state(new Map());
  const crossings = $derived.by(() => {
    const crossings: Map<number, Map<number, number>> = new Map();

    for (const [depth, nodes] of connections) {
      const dcrossings = crossings.getOrInsertComputed(depth, () => new Map());

      for (const [source, target] of nodes)
        for (let i = source + 1; i < target; i++)
          dcrossings.set(
            i,
            dcrossings.getOrInsert(i, nodes.get(i) === undefined ? 0 : 1) + 1,
          );
    }

    return crossings;
  });
  const outputs = $derived.by(() => {
    if (connections.size === 0) return inputs;

    const output = Array.from(inputs);
    for (let d = 0; d < depth; d++) {
      const nodes = connections.get(d);
      if (nodes === undefined) continue;
      for (let i = 0; i < output.length; i++) {
        const connection = nodes.get(i);
        if (connection !== undefined && output[connection] < output[i])
          [output[i], output[connection]] = [output[connection], output[i]];
      }
    }
    return output;
  });

  function connect(depth: number, source: number, target: number) {
    if (source > target) [source, target] = [target, source];
    const nodes = connections.getOrInsertComputed(depth, () => new Map());

    // If either node is already targeted, break its connection
    for (const n of [source, target]) {
      const deleteme_source = nodes.entries().find(([_, d]) => d === n)?.[0];
      if (deleteme_source !== undefined) nodes.delete(deleteme_source);
    }

    nodes.delete(target);
    if (source !== target) nodes.set(source, target);
  }

  function connection_style(depth: number, row: number) {
    const colors = [
      'bg-black',
      'bg-black',
      'bg-red-700',
      'bg-yellow-600',
      'bg-teal-700',
      'bg-orange-600',
    ];

    const n_crossings = crossings.get(depth)?.get(row) ?? 0;
    if (n_crossings >= colors.length)
      console.warn(
        `node (${depth}, ${row}) has more crossings than there are available colors; wrapping around`,
      );
    const color = colors[n_crossings % colors.length];
    const z_index = n_crossings == 0 ? '-z-1' : `z-${n_crossings}`;

    return `${color} ${z_index}`;
  }

  function shuffle<T>(array: T[]) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
  }
</script>

<div class="grid h-screen w-screen grid-rows-5">
  <div class="row-start-1 row-end-3 flex items-center justify-center">
    <div class="text-center text-lg">
      <h3>click two nodes at the same depth to make a connection</h3>
      <h3>stacked connections are color-coded</h3>
    </div>
  </div>
  <div
    class="
      row-3
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
                bg-gray-200
                ${source?.[0] === depth && source?.[1] === row ? 'border-2 border-blue-500' : 'border border-black'}
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
                    cursor-default
                    ${connection_style(depth, row)}
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
  <div class="-row-2 flex items-end justify-center gap-3">
    <NumAdjust label="Depth" bind:value={depth} min={1} />
    <NumAdjust label="Inputs" bind:value={inputs.length} min={2} />
    <Button onclick={() => shuffle(inputs)}>Shuffle inputs</Button>
    <Button onclick={() => connections.clear()}>Clear connections</Button>
  </div>
  <p class="-row-1 my-1.5 text-center">
    made by <Link href="https://justdeeevin.dev">devin droddy</Link>
    (<Link
      href="https://github.com/justdeeevin/comparator-network-sim"
      target="_none"
    >
      source
    </Link>)
  </p>
</div>
