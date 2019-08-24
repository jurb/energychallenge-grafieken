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
  const activeBars = ["Week 1", ""];
  const padding = { top: 250, right: 0, bottom: 120, left: 120 };
  let width = 800;
  let height = 800;

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

  onMount(() => {
    var svg = document.querySelector("svg");
    var canvas = document.createElement("canvas");
    canvas.height = height;
    canvas.width = width;
    canvg(canvas, svg.parentNode.innerHTML.trim());
    console.log(svg.parentNode.innerHTML.trim());
    console.log(canvg(canvas, svg.parentNode.innerHTML.trim()));
    // var dataURL = canvas.toDataURL("image/png");
    // var data = atob(dataURL.substring("data:image/png;base64,".length)),
    //   asArray = new Uint8Array(data.length);

    // for (var i = 0, len = data.length; i < len; ++i) {
    //   asArray[i] = data.charCodeAt(i);
    // }

    // var blob = new Blob([asArray.buffer], { type: "image/png" });
    // saveAs(blob, "export_" + Date.now() + ".png");
  });
</script>

<style>
  svg {
    position: relative;
    width: 800px;
    height: 800px;
    border: 1px solid black;
  }
</style>

<div style="width: {width}; height: {height}">
  <svg id="svg">

    <!-- annotations -->
    <g class="annotations-manual">
      <path
        d="M {padding.left - 25}
        {y(dataAll[0].value) - 5} C 100 50 {padding.left - 20}
        {padding.left - 20}
        {width / 2 - 100}
        {padding.top - 80}"
        style="stroke:grey;stroke-width:1;fill:transparent" />

      <g transform="translate({width / 2 - 100},{padding.top - 130})">
        <rect
          class="annotation-note-bg"
          width="200"
          height="100"
          x="0"
          y="0"
          fill="goldenrod"
          fill-opacity="0.2" />
      </g>

      <g
        class="annotation-note-content"
        transform="translate({width / 2}, {padding.top - 90})">
        <text class="annotation-note-label" dx="0" y="30" fill="grey">
          <tspan x="0" dy="0">Week 1</tspan>
        </text>
        <text class="annotation-note-title" fill="grey" font-weight="bold">
          <tspan x="0" dy="0">⚡ {int(dataTestweek.value)} kWh ⚡</tspan>
        </text>
      </g>
    </g>

    <!-- bars -->
    <g class="bars">
      <rect
        x={padding.left - barWidth + 21}
        y={y(dataTestweek.value)}
        width={barWidth - 4}
        height={height - padding.bottom - y(dataTestweek.value)}
        class="testweek" />
      <text
        class="testweek"
        x={padding.left - 27}
        y={y(dataTestweek.value) + 35}>
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
        <text x={padding.left - 42} y={-padding.bottom + 35}>
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
<!-- <button on:click={() => tweenedData.set(data2)}>data2</button> -->
<!-- {activeBar('1990')} -->
<!-- {annotationGen()} -->
<!-- {dataTestweek} {y(dataTestweek)} {barWidth} -->
<!-- {dataMax} {dataMax} {linegen(dataIncoming)} -->
<!-- {width} -->
