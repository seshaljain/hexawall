<script>
  import { onMount } from "svelte";

  const HEX_SIZE = 48;
  let rows = 0;
  let cols = 0;
  let cellStates = [];
  let darkMode = true;

  function updateGridSize() {
    const hexWidth = HEX_SIZE * 2;
    const hexHeight = Math.sqrt(3) * HEX_SIZE;
    cols = Math.ceil(window.innerWidth / (HEX_SIZE * 1.5)) + 2;
    rows = Math.ceil(window.innerHeight / hexHeight) + 2;
    if (cellStates.length !== rows || cellStates[0]?.length !== cols) {
      cellStates = Array.from({ length: rows }, (_, r) =>
        Array.from({ length: cols }, (_, c) => cellStates[r]?.[c] || false),
      );
    }
  }

  // Flat-topped hexagons, non-overlapping, adjacent
  function hexCenter(row, col) {
    const hexHeight = Math.sqrt(3) * HEX_SIZE;
    const x = col * (HEX_SIZE * 1.5);
    const y = row * hexHeight + (col % 2) * (hexHeight / 2);
    return { x, y };
  }

  function hexPoints(cx, cy, size = HEX_SIZE) {
    return Array.from({ length: 6 }, (_, i) => {
      const angle = (Math.PI / 3) * i;
      return [cx + size * Math.cos(angle), cy + size * Math.sin(angle)];
    });
  }

  // For the counter hexagon
  function counterHexPoints(cx = 36, cy = 36, size = HEX_SIZE * 0.6) {
    return hexPoints(cx, cy, size)
      .map(([x, y]) => `${x},${y}`)
      .join(" ");
  }

  function toggleCell(row, col) {
    cellStates[row][col] = !cellStates[row][col];
    cellStates = [...cellStates];
  }

  // Count active tiles
  $: activeCount = cellStates.flat().filter(Boolean).length;

  onMount(() => {
    updateGridSize();
    window.addEventListener("resize", updateGridSize);
    return () => window.removeEventListener("resize", updateGridSize);
  });
</script>

<div
  class={`fixed inset-0 overflow-hidden ${darkMode ? "bg-gray-900" : "bg-yellow-50"}`}
>
  <svg class="w-full h-full block">
    {#each Array(rows) as _, row}
      {#each Array(cols) as _, col}
        {#key `${row}-${col}`}
          <polygon
            points={hexPoints(hexCenter(row, col).x, hexCenter(row, col).y)
              .map(([px, py]) => `${px},${py}`)
              .join(" ")}
            fill={cellStates[row][col]
              ? darkMode
                ? "var(--color-slate-500)"
                : "var(--color-yellow-300)"
              : darkMode
                ? "var(--color-slate-900)"
                : "var(--color-yellow-50)"}
            stroke={darkMode
              ? "var(--color-gray-800)"
              : "var(--color-yellow-400)"}
            stroke-width="8"
            on:click={() => toggleCell(row, col)}
            class="transition-colors duration-100 cursor-pointer"
          />
        {/key}
      {/each}
    {/each}
  </svg>
  <div
    class="absolute top-4 right-6 z-10 select-none flex flex-col items-center justify-center gap-2"
  >
    <!-- Tile Counter Hexagon -->
    <svg width="72" height="72" viewBox="0 0 72 72" class="drop-shadow-lg mb-1">
      <polygon
        points={counterHexPoints(36, 36, HEX_SIZE * 0.6)}
        fill={darkMode ? "var(--color-slate-800)" : "#fde047"}
        stroke={darkMode ? "var(--color-yellow-300)" : "#ca8a04"}
        stroke-width="4"
      />
      <text
        x="36"
        y="36"
        text-anchor="middle"
        dominant-baseline="central"
        font-size="1.3rem"
        fill={darkMode ? "var(--color-yellow-300)" : "#92400e"}
        font-family="monospace"
        font-weight="bold"
      >
        {activeCount}
      </text>
    </svg>
    <!-- Mode Toggle: Small Circle with Minimal Icon -->
    <button
  class="p-0 bg-transparent border-0 outline-none"
  on:click={() => (darkMode = !darkMode)}
  aria-label="Toggle dark/light mode"
  title="Toggle dark/light mode"
  style="line-height:0"
>
  <svg width="44" height="44" viewBox="0 0 44 44" class="drop-shadow">
    <circle
      cx="22"
      cy="22"
      r="20"
      fill={darkMode ? "var(--color-slate-800)" : "var(--color-yellow-50)"}
      stroke={darkMode
        ? "var(--color-yellow-300)"
        : "var(--color-yellow-400)"}
      stroke-width="0"
    />
    {#if darkMode}
      <!-- Sun Icon: larger center, shorter rays, yellow-300 -->
      <circle cx="22" cy="22" r="5" fill="#fde047"/>
      <g stroke="#fde047" stroke-width="1.5" stroke-linecap="round">
        <line x1="22" y1="13" x2="22" y2="10"/>
        <line x1="22" y1="31" x2="22" y2="34"/>
        <line x1="13" y1="22" x2="10" y2="22"/>
        <line x1="31" y1="22" x2="34" y2="22"/>
        <line x1="16.5" y1="16.5" x2="14.5" y2="14.5"/>
        <line x1="27.5" y1="27.5" x2="29.5" y2="29.5"/>
        <line x1="16.5" y1="27.5" x2="14.5" y2="29.5"/>
        <line x1="27.5" y1="16.5" x2="29.5" y2="14.5"/>
      </g>
    {:else}
      <!-- Minimal Moon Icon -->
      <path
        d="M27 22a7 7 0 1 1-7-7A5 5 0 0 0 27 22Z"
        fill="var(--color-yellow-300)"
      />
    {/if}
  </svg>
</button>
  </div>
</div>
