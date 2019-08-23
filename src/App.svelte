<script>
  import { onMount } from "svelte";
  import { scaleLinear, scaleBand } from "d3-scale";
  import { ticks, max, sum } from "d3-array";
  import { select } from "d3-selection";
  import { format } from "d3-format";
  import { line, curveBasis } from "d3-shape";

  // Settings
  const activeBars = ["Week 1", "Week 2"];
  const padding = { top: 120, right: 50, bottom: 90, left: 120 };
  let width = 900;
  let height = 500;

  // Helpers
  const activeBar = name => activeBars.includes(name);
  const int = format(",.0f");
  const params = new URLSearchParams(window.location.search);

  // Default data, not used yet
  export let dataDefault = [
    { name: "Vorig jaar", value: 20 },
    { name: "Week 1", value: 20 },
    { name: "Week 2", value: 20 },
    { name: "Week 3", value: 20 },
    { name: "Week 4", value: 20 }
  ];

  const barNames = ["Vorig jaar", "Week 1", "Week 2", "Week 3", "Week 4"];

  const urlData = params
    .get("data")
    .split(",")
    .map(Number);

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
    .padding(0.3);

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
    width: 100%;
    height: 500px;
    border: 1px solid black;
  }
</style>

<div style="margin:100px" />

<div class="chart" bind:clientWidth={width} bind:clientHeight={height}>

  <svg id="svg">

    <g class="annotations-manual" style=" border: 1px solid black; ">

      <!-- <g class="pointer" transform="translate({padding.top}, {padding.left})">
        <path
          d="M {padding.left - 20}
          {y(dataAll[0].value) - 5} C {padding.left - 20} 80 {padding.left - 20}
          {padding.left - 20}
          {width - padding.left - 30} 40"
          style="stroke:grey;stroke-width:1;fill:transparent" />
      </g> -->

      <g
        class="annotation-note-content"
        transform="translate({width - padding.left}, 25)">
        <text class="annotation-note-label" dx="0" y="20" fill="grey">
          <tspan x="0" dy="0.8em">Week 1</tspan>
        </text>
        <text class="annotation-note-title" fill="grey" font-weight="bold">
          <tspan x="0" dy="0.8em">⚡ {int(dataTestweek.value)} kWh ⚡</tspan>
        </text>
      </g>
    </g>

    <!-- y axis -->
    <g class="axis y-axis" transform="translate(0,{padding.top})">
      <g
        class="tick"
        transform="translate(0, {y(dataTestweek.value) - padding.top - 1})">
        <line x2="100%" />
      </g>
    </g>

    <!-- x axis -->
    <g class="axis x-axis">
      <g class="tick" transform="translate(15,{height})">
        <text x={padding.left - 42} y={-padding.bottom + 20}>
          {dataTestweek.name}
        </text>
      </g>

      {#each data as dataPoint, i}
        <g class="tick" transform="translate({x(dataPoint.name)},{height})">
          <text x={barWidth / 2.15} y={-padding.bottom + 20}>
            {dataPoint.name}
          </text>
        </g>
      {/each}
    </g>

    <g class="bars">
      <rect
        x={padding.left - barWidth + 10}
        y={y(dataTestweek.value)}
        width={barWidth - 4}
        height={height - padding.bottom - y(dataTestweek.value)}
        class="testweek" />
      <text
        class="testweek"
        x={padding.left - 27}
        y={y(dataTestweek.value) + 18}>
        {int(dataTestweek.value)} kWh
      </text>

      {#each data as dataPoint, i}
        <rect
          x={x(dataPoint.name)}
          y={y(dataPoint.value)}
          width={barWidth - 4}
          height={height - padding.bottom - y(dataPoint.value)}
          class={activeBar(dataPoint.name) ? 'active' : 'inactive'} />
        <text
          class={activeBar(dataPoint.name) ? 'active' : 'inactive'}
          x={x(dataPoint.name) + x.bandwidth() / 2 - 3}
          y={y(dataPoint.value) + 18}>
          {activeBar(dataPoint.name) ? `${int(dataPoint.value)} kWh` : '?'}
        </text>
      {/each}
    </g>

    <g id="annotations" />
  </svg>
</div>
<!-- <button on:click={() => tweenedData.set(data2)}>data2</button> -->
<!-- {activeBar('1990')} -->
<!-- {annotationGen()} -->
<!-- {dataTestweek} {y(dataTestweek)} {barWidth} -->
<!-- {dataMax} {dataMax} {linegen(dataIncoming)} -->
