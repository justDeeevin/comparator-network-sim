<script lang="ts">
  import NumAdjust from '$lib/components/num-adjust.svelte';
  import Button from '$lib/components/button.svelte';
  import NumInput from '$lib/components/num-input.svelte';
  import Link from '$lib/components/link.svelte';
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
  /**
    depth->row->how many crossings
  */
  const crossings = $derived.by(() => {
    const out: Map<number, Map<number, number>> = new Map();

    for (const [depth, nodes] of connections)
      for (const [k, v] of nodes) {
        const map = out.getOrInsertComputed(depth, () => new Map());
        for (let i = 0; i < v - k - 1; i++)
          map.set(k + i + 1, map.getOrInsert(k + i + 1, 0) + 1);
      }

    console.debug(out);

    return out;
  });

  const outputs = $derived.by(() => {
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

  function shuffled<T>(array: T[]) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }

    return array;
  }
</script>

<div class="grid h-screen w-screen grid-rows-5">
  <div class="row-start-1 row-end-3 flex items-center justify-center">
    <h3 class="text-lg">
      click two nodes at the same depth to make a connection
    </h3>
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
  <div class="-row-2 flex items-end justify-center gap-3">
    <NumAdjust label="Depth" bind:value={depth} min={1} />
    <NumAdjust label="Inputs" bind:value={inputs.length} min={2} />
    <Button onclick={() => (inputs = shuffled(inputs))}>Shuffle inputs</Button>
    <Button onclick={() => (connections = new Map())}>Clear connections</Button>
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
