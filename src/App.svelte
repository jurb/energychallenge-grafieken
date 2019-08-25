<script>
  import { onMount } from "svelte";
  import { scaleLinear, scaleBand } from "d3-scale";
  import { ticks, max, sum } from "d3-array";
  import { select } from "d3-selection";
  import { format } from "d3-format";
  import { line, curveBasis } from "d3-shape";
  import saveAs from "file-saver";
  import canvg from "canvg";

  // Settings
  const padding = { top: 200, right: 0, bottom: 120, left: 120 };
  let width = 600;
  let height = 600;

  // Helpers
  const activeBar = name => activeBars.includes(name);
  const int = format(",.0f");

  // Default data, not used yet
  // export let dataDefault = [
  //   { name: "Weekverbruik vorig jaar", value: 20 },
  //   { name: "Week 1", value: 20 },
  //   { name: "Week 2", value: 20 },
  //   { name: "Week 3", value: 20 },
  //   { name: "Week 4", value: 20 }
  // ];

  const barNames = [
    "Weekverbruik vorig jaar",
    "Week 1",
    "Week 2",
    "Week 3",
    "Week 4"
  ];

  // URL paramater parsing and settings
  const params = new URLSearchParams(window.location.search);
  const urlData = params
    .get("data")
    .split(",")
    .map(Number);

  const urlWeeks =
    params.get("weeks") != null ? params.get("weeks").split(",") : "";

  const activeBars = urlWeeks;

  const urlWeek = urlData.length - 1;

  $: dataAll = urlData.map((el, i) => ({
    name: barNames[i],
    value: el
  }));

  $: data = dataAll.slice(1, 5);
  $: dataTestweek = dataAll[0];

  // Data variables
  $: dataSum = sum(data, d => d.value);
  $: dataMax = max(dataAll, d => d.value);

  const linegen = line()
    .x(d => x(d.name))
    .y(d => y(d.value));

  const linegenTestweek = line()
    .x(d => 60)
    .y(d => y(d.value));

  // Scales
  $: x = scaleBand()
    .domain(data.map(d => d.name))
    .range([padding.left + 60, width - padding.right]) // Offset of 60 pixels, manual first bar with more padding
    .padding(0.25);

  $: y = scaleLinear()
    .domain([0, dataMax])
    // .nice(5)
    .range([height - padding.bottom - 25, padding.top]); // Bars have a min height of 25px, for labels

  $: yTicks = y.ticks(5);
  $: barWidth = x.bandwidth();
</script>

<style>
  svg {
    position: relative;
    width: 600px;
    height: 600px;
    border: 1px solid lightgrey;
  }
</style>

<div style="width: {width}; height: {height}">
  <svg id="svg">

    <!-- annotations -->
    <g class="annotations-manual">

      <!-- arrow for testweek -->
      <path
        d="M 100 {y(dataAll[0].value) - 5} C 40 100 100 110 100 110"
        class="arrow" />
      <g transform="translate(100,50)">
        <path d="M -9 67 c 2-3 6-4 9-7-3-1-9-3-11-6" class="arrow" />
      </g>
    </g>

    <!-- arrow for week 1 -->
    <path
      d="M {x(data[0].name) + barWidth / 2}
      {y(data[0].value) - 5} C 270 200 270 270 290 170"
      class="arrow" />
    <g transform="translate(230,170) rotate(-80 -5 -5)">
      <path class="arrow" d="M -9 67 c 2-3 6-4 9-7-3-1-9-3-11-6" />
    </g>

    <rect
      class="annotation-note-bg"
      width="400"
      height="110"
      x="100"
      y="60"
      fill="goldenrod"
      fill-opacity="0.2" />

    <g class="annotation-note-content" transform="translate({width / 2}, 86)">
      <text class="annotation-note-label">
        <tspan x="0" dy="0" style="font-size:3em">⚡</tspan>
      </text>
      <text class="annotation-note-label">
        <tspan x="0" dy="-20" style="font-size:0.7em">❌</tspan>
      </text>
      <text class="annotation-note-label">
        <tspan x="0" dy="-20" style="font-size:0.7em">✅</tspan>
      </text>
      <text class="annotation-note-label" dx="0" y="30">
        <tspan x="0" dy="-5">Mooi op schema. Ga zo door!</tspan>
      </text>
      <text
        class="annotation-note-label"
        dx="0"
        y="30"
        style="font-size:.9em; font-family: 'AvenirNextLTW01-Regular'">
        <tspan x="0" dy="20">
          Jullie zitten nu 7% onder het jullie gebruik van vorig jaar.
        </tspan>
      </text>
      <text
        class="annotation-note-label"
        dx="0"
        y="30"
        style="font-size:.9em; font-family: 'AvenirNextLTW01-Regular'">
        <tspan x="0" dy="40">In Week 2 gebruikten jullie 5% minder.</tspan>
      </text>
      <!-- <text class="annotation-note-title">
        <tspan x="0" dy="0">⚡ {int(dataTestweek.value)} kWh ⚡</tspan>
      </text> -->
    </g>

    <!-- bars -->
    <g class="bars">
      <rect
        x={padding.left - barWidth + 25}
        y={y(dataTestweek.value)}
        width={barWidth - 4}
        height={height - padding.bottom - y(dataTestweek.value)}
        class="testweek" />
      <text
        class="testweek"
        x={padding.left - 14}
        y={y(dataTestweek.value) + 35}>
        {int(dataTestweek.value)} kWh
      </text>

      {#each data as dataPoint, i}
        <rect
          x={x(dataPoint.name)}
          y={y(dataPoint.value)}
          width={barWidth - 4}
          height={height - padding.bottom - y(dataPoint.value)}
          class={activeBar(dataPoint.name) ? 'known' : 'unknown'} />
        <text
          class={activeBar(dataPoint.name) ? 'known' : 'unknown'}
          x={x(dataPoint.name) + barWidth / 2 - 3}
          y={y(dataPoint.value) + 35}>
          {activeBar(dataPoint.name) ? `${int(dataPoint.value)} kWh` : '?'}
        </text>
      {/each}
    </g>

    <!-- y axis line -->
    <g class="axis y-axis" transform="translate(0,{padding.top})">
      <g
        class="tick"
        transform="translate(0, {y(dataTestweek.value) - padding.top - 1})">
        <line x2={width} />
      </g>
    </g>

    <!-- x axis bar labels -->
    <g class="axis x-axis">
      <g class="tick" transform="translate(15,{height})">
        <text x={padding.left - 30} y={-padding.bottom + 35}>
          {dataTestweek.name}
        </text>
      </g>
      {#each data as dataPoint, i}
        <g class="tick" transform="translate({x(dataPoint.name)},{height})">
          <text x={barWidth / 2.15} y={-padding.bottom + 35}>
            {dataPoint.name}
          </text>
        </g>
      {/each}
    </g>
  </svg>
</div>
{urlWeek}
