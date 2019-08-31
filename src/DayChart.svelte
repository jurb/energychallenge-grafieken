<script>
  import { scaleLinear, scaleBand } from "d3-scale";
  import { ticks, max, sum, mean } from "d3-array";
  import { select } from "d3-selection";
  import { format } from "d3-format";
  import { line, curveBasis } from "d3-shape";

  // props
  export let urlData;
  export let gezin;
  export let kind;

  const data = urlData;

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
    .domain(data.map(d => d.name))
    .range([padding.left, width - padding.right]) // Offset of 60 pixels, manual first bar with more padding
    .padding(0.25);

  $: y = scaleLinear()
    .domain([0, dataMax])
    .range([height - padding.bottom - 25, padding.top]); // Bars have a min height of 25px, for labels

  $: yTicks = y.ticks(5);
  $: barWidth = x.bandwidth();

  // Settings
  const padding = { top: 240, right: 15, bottom: 120, left: 0 };
  let width = 600;
  let height = 600;
</script>

<style>

</style>

<div style="width: {width}; height: {height}">

  <svg id="svg">

    <!-- annotations -->
    <g class="annotations-manual">

      <!-- arrows -->
      <g transform="translate(45,0)">
        <path
          d="M 229.5, 247 C 230, 250, 224, 290, 162, 320"
          class="arrow-red" />
        <g transform="translate(179,215.7) rotate(-60,0,0)">
          <path d="M -9 67 c 2-3 6-4 9-7-3-1-9-3-11-6" class="arrow-red" />
        </g>
      </g>
      <text class="annotation-note-title" x="165" y="310">Blijf onder</text>
      <text class="annotation-note-title" x="165" y="330">deze lijn!</text>

      <path
        d="M {x(dataMax) + barWidth / 2 - 3}
        {y(dataMax)} C 210 210 210 210 {x(dataMax) + barWidth / 2} 170"
        class="arrow" />
      <g transform="translate(270.6,{y(dataMax) + 0}) rotate(95,0,0)">
        <path d="M -9 67 c 2-3 6-4 9-7-3-1-9-3-11-6" class="arrow" />
      </g>

      <!-- feedback area -->
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

      </g>
      <!-- bars -->
      <g class="bars">
        {#each data as dataPoint, i}
          <rect
            x={x(dataPoint)}
            y={y(dataPoint)}
            width={20}
            height={height - padding.bottom - y(dataPoint)} />
          <text x={x(dataPoint) + barWidth / 2 - 5} y={y(dataPoint) + 35}>

            {#if kind == 'gas'}
              {nlformat(dataPoint)} m
              <tspan baseline-shift="super" font-size="8" dx="-2">3</tspan>
            {:else if kind == 'stroom'}{nlformat(dataPoint)} kWh{/if}
          </text>
        {/each}

        <!-- x axis bar labels -->
        <g class="axis x-axis">
          {#each data as dataPoint, i}
            <g class="tick" transform="translate({x(dataPoint.name)},{height})">
              <text x={barWidth / 2.15} y={-padding.bottom + 25}>
                {dataPoint.name}
              </text>
            </g>
          {/each}
        </g>
        <!-- y axis line
    <g class="axis y-axis" transform="translate(0,{padding.top})">
      <g
        class="tick"
        transform="translate(0, {y(testAmount) - padding.top + 1})">
        <line x1={x2(dataLeft[0].name) + barWidth} x2={width} />
      </g>
    </g> -->
      </g>
    </g>
  </svg>
</div>

<h3>{gezin}</h3>
