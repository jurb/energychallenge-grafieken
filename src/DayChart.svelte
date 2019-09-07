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

  const data = urlData;
  const hours = range(0, 24);

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
    .range([height - padding.bottom, padding.top]);

  $: yTicks = y.ticks(6);
  $: barWidth = x.bandwidth();

  // Settings
  const padding = { top: 240, right: 15, bottom: 120, left: 30 };
  let width = 600;
  let height = 600;
</script>

<style>

</style>

<div style="width: {width}; height: {height}">

  <svg id="svg">
    <!-- bars -->
    <g class="bars">
      {#each data as dataPoint, i}
        <rect
          x={x(i)}
          y={y(dataPoint)}
          width={20}
          height={height - padding.bottom - y(dataPoint)}
          class="testweek" />
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
      {#each hours as hour}
        {#if hour % 2 == 0}
          <g class="tick" transform="translate({x(hour)},{height})">
            <text
              x={barWidth / 2.15}
              y={-padding.bottom + 10}
              style="font-size: 5px">
              |
            </text>
            <text x={barWidth / 2.15} y={-padding.bottom + 25}>{hour}:00</text>
            {#if hour == 0}
              <text x={barWidth / 2.15} y={-padding.bottom + 35}>uur</text>
            {/if}
          </g>
        {/if}
      {/each}
    </g>
    <g class="axis y-axis" transform="translate(0,{padding.top})">
      {#each yTicks as tick}
        <g
          class="tick ytick"
          transform="translate(-20, {y(tick) - padding.top + 1})">
          <line x1={padding.left} x2={width} />
          <text x={padding.left} y="-4">
            {tick} {tick === yTicks[yTicks.length - 1] ? ` ${unit}` : ''}
          </text>

        </g>
      {/each}

    </g>
  </svg>
</div>

<h3>{gezin}</h3>
{data}
<br />
{dataMax}
<br />
{y(25)} {x(25)}
<br />
{yTicks[yTicks.length - 1]}
<br />
{hours}
