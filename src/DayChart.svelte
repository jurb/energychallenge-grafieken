<script>
  import { scaleLinear, scaleBand, scaleTime } from "d3-scale";
  // import { scaleTime } from "d3-scale";
  import { ticks, max, sum, mean, range } from "d3-array";
  import { select } from "d3-selection";
  import { format } from "d3-format";
  // import { axisBottom, axisRight } from "d3-axis";
  // import { line, curveBasis } from "d3-shape";
  // import { timeFormat } from "d3-time-format";
  // import { onMount } from "svelte";

  // props
  export let urlData;
  export let gezin;
  export let kind;
  export let date;

  const data = urlData;
  const gasRange = range(0, 24);
  const stroomRange = range(0, 96);

  // Helpers
  const int = format(",.0f");
  const nlformat = n => Number(parseFloat(n).toFixed(1)).toLocaleString("nl");

  // URL paramater parsing and settings
  const icon = kind == "stroom" ? "⚡️" : "🔥";
  const unit = kind == "stroom" ? "kWh" : "m3";

  // Data variables
  const dataSum = sum(data, d => d);
  const dataMax = max(data, d => d);
  const dataMean = mean(data, d => d);

  // Scales
  $: x = scaleBand()
    .domain(data.map((d, i) => i))
    .range([padding.left, width - padding.right])
    .padding(0.1);

  $: y = scaleLinear()
    .domain([0, dataMax])
    .range([height - padding.bottom, padding.top])
    .nice();

  $: yTicks = y.ticks(6);
  $: barWidth = x.bandwidth();

  // Settings
  const padding = { top: 240, right: 15, bottom: 120, left: 50 };
  let width = 600;
  let height = 600;
</script>

<style>

</style>

<div style="width: {width}; height: {height}">

  <svg id="svg">

    <!-- annotations -->
    <g class="annotations-manual">

      <!-- arrow -->
      <!-- <path d="M 300 165 C 330 250 440 140 450 242" class="arrow" />
      <g transform="translate(510,250) rotate(95,0,0)">
        <path d="M -9 67 c 2-3 6-4 9-7-3-1-9-3-11-6" class="arrow" />
      </g> -->

      <!-- background -->
      <rect
        class="annotation-note-bg"
        width="400"
        height="110"
        x="100"
        y="60"
        fill="goldenrod"
        fill-opacity="0.2" />

      <!--  Icon block -->
      <g class="annotation-note-content" transform="translate({width / 2}, 86)">
        {#if kind == 'stroom'}
          <text class="annotation-icon" x="-20" dy="5" style="font-size: 3.5em">
            {icon}
          </text>
        {/if}
        {#if kind == 'gas'}
          <text class="annotation-icon" x="-19" dy="-20">{icon}</text>
        {/if}

        <g id="feedback" class="active">
          <text class="annotation-note-title" dx="0" y="10">
            Jullie {kind} verbruik op {date}
          </text>
          {#if kind == 'gas'}
            <text class="annotation-note-label" dx="0" y="40">
              <tspan>Kunnen jullie zien op welke</tspan>
            </text>
            <text class="annotation-note-label" dx="0" y="40">
              <tspan y="60" x="0">
                momenten jullie onder de douche stonden?
              </tspan>
            </text>
          {:else}
            <text class="annotation-note-label" dx="0" y="40">
              <tspan>Kunnen jullie zien op welke</tspan>
            </text>
            <text class="annotation-note-label" dx="0" y="40">
              <tspan y="60" x="0">momenten jullie stroom gebruikten?</tspan>
            </text>
          {/if}
        </g>
      </g>
    </g>

    <!-- bars -->
    <g class="bars">
      {#each data as dataPoint, i}
        <rect
          x={x(i)}
          y={y(dataPoint)}
          width={kind === 'gas' ? 16 : 4}
          height={height - padding.bottom - y(dataPoint)}
          class="daybar" />
        <!-- <text x={x(i) + barWidth / 2 - 5} y={y(dataPoint) + 35}>

          {#if kind == 'gas'}
            {nlformat(dataPoint)} m
            <tspan baseline-shift="super" font-size="8" dx="-2">3</tspan>
          {:else if kind == 'stroom'}{nlformat(dataPoint)} kWh{/if}
        </text> -->
      {/each}
    </g>

    <!-- axis bar labels -->
    <g class="axis x-axis-day">
      {#if kind == 'gas'}
        <!-- one observation every hour -->
        {#each gasRange as observation}
          <g class="tick" transform="translate({x(observation) - 2},{height})">
            <text
              x={barWidth / 2.15}
              y={-padding.bottom + 6}
              style="font-size: 5px">
              |
            </text>
          </g>
          {#if observation == 0}
            <g
              class="tick"
              transform="translate({x(observation) - 2},{height})">
              <text x={barWidth / 2.15} y={-padding.bottom + 25}>
                {observation}:00
              </text>
              <text x={barWidth / 2.15} y={-padding.bottom + 37}>uur</text>
            </g>
          {:else}
            <g
              class="tick"
              transform="translate({x(observation) - 2},{height})">
              <text x={barWidth / 2.15} y={-padding.bottom + 25}>
                {observation}
              </text>
            </g>
          {/if}
        {/each}
      {/if}

      {#if kind == 'stroom'}
        <!-- one observation every 15 minutes -->
        {#each stroomRange as observation}
          <g class="tick" transform="translate({x(observation) - 2},{height})">
            <text
              x={barWidth / 2.15}
              y={-padding.bottom + 6}
              style="font-size: 5px">
              |
            </text>
          </g>
          {#if observation == 0}
            <g
              class="tick"
              transform="translate({x(observation) - 2},{height})">
              <text x={barWidth / 2.15} y={-padding.bottom + 25}>
                {observation}:00
              </text>
              <text x={barWidth / 2.15} y={-padding.bottom + 37}>uur</text>
            </g>
          {:else if observation % 4 == 0}
            <g
              class="tick"
              transform="translate({x(observation) - 2},{height})">
              <text x={barWidth / 2.15} y={-padding.bottom + 25}>
                {observation / 4}
              </text>
            </g>
          {/if}
        {/each}
      {/if}

    </g>
    <g class="axis y-axis" transform="translate(0,{padding.top})">
      {#each yTicks as tick, i}
        <g
          class="tick ytick"
          transform="translate(-20, {y(tick) - padding.top + 1})">
          {#if i > 0}
            <line class="dayline" x1={padding.left - 10} x2={width} />
          {/if}
          <text x={padding.left - 10} y="-4">
            {#if tick === yTicks[yTicks.length - 1] && kind == 'stroom'}
              {tick} kWh
            {:else if tick === yTicks[yTicks.length - 1] && kind == 'gas'}
              {tick} m
              <tspan baseline-shift="super" font-size="10" dx="-3">3</tspan>
            {:else}{tick}{/if}
          </text>

        </g>
      {/each}

    </g>
  </svg>
</div>

<h3>{gezin}</h3>
