<script lang="ts">
  type Circle = {
    id: string;
    cx: number;
    cy: number;
    r: number;
  };

  type Status = 'drawing' | 'editing';

  let status = $state<Status>('drawing');
  let circles = $state<Circle[]>([]);
  let selectedCircle = $state<Circle | null>(null);

  let snapshots: Circle[][] = [];
  let history = $state(-1);

  function undo() {
    selectedCircle = null;
    status = 'drawing';
    circles = snapshots[--history];
  }
  function redo() {
    selectedCircle = null;
    status = 'drawing';
    circles = snapshots[++history];
  }
  function snapshot() {
    history++;
    snapshots.push($state.snapshot(circles));
  }

  function testlog() {
    console.log($state.snapshot(snapshots));
    console.log($state.snapshot(history));
    console.log($state.snapshot(selectedCircle));
  }

  function drawCircle(event: MouseEvent) {
    const svgEl = event.target as SVGElement;
    const { left, top } = svgEl.getBoundingClientRect();

    const circle: Circle = {
      id: crypto.randomUUID(),
      cx: +(event.clientX - left).toFixed(),
      cy: +(event.clientY - top).toFixed(),
      r: 25,
    };

    circles.push(circle);
    selectedCircle = circles[circles.length - 1];
    status = 'editing';
    snapshot();
  }
</script>

<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_noninteractive_element_interactions -->
<main
  class="flex flex-col pt-20 items-center bg-neutral-700 text-neutral-100 h-screen"
  onclick={(event) => {
    event.stopPropagation();
    if (event.target === event.currentTarget) {
      if (selectedCircle) {
        status = 'drawing';
        selectedCircle = null;
        snapshot();
      }
    }
  }}
>
  <div class="flex items-center mb-2 gap-2">
    <button
      onclick={undo}
      disabled={history === -1}
      class="px-3 py-2 bg-teal-700 hover:bg-teal-600 transition-colors rounded font-bold text-sm disabled:hover:bg-teal-700 disabled:text-neutral-400 disabled:cursor-not-allowed"
    >
      Undo
    </button>
    <button
      onclick={redo}
      disabled={history === snapshots.length - 1}
      class="px-3 py-2 bg-teal-700 hover:bg-teal-600 transition-colors rounded font-bold text-sm disabled:hover:bg-teal-700 disabled:text-neutral-400 disabled:cursor-not-allowed"
    >
      Redo
    </button>
    <button onclick={testlog}>log</button>
  </div>
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <svg
    onclick={drawCircle}
    viewBox="0 0 600 400"
    class="w-[600px] h-[400px] border-2 border-neutral-500 bg-neutral-900"
  >
    {#each circles as circle}
      <circle
        {...circle}
        stroke="#fff"
        fill={selectedCircle?.id === circle.id ? '#0f0' : '#000'}
        stroke-width="2"
        onclick={(event) => {
          event.stopPropagation();
          selectedCircle = circle;
          status = 'editing';
        }}
      ></circle>
    {/each}
  </svg>
  {#if status === 'editing' && selectedCircle}
    <div class="flex mt-2 flex-col items-center">
      <h2>Adjust diameter of circle at ({selectedCircle.cx}, {selectedCircle.cy})</h2>
      <input type="range" step="1" min="0" max="100" bind:value={selectedCircle.r} />
    </div>
  {/if}
</main>

<style>
</style>
