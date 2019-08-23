<script>
  import { onMount } from "svelte";
  import { scaleLinear, scaleBand } from "d3-scale";
  import { ticks, max, sum } from "d3-array";
  import { select } from "d3-selection";
  import { format } from "d3-format";
  import { line, curveBasis } from "d3-shape";
  import {
    annotation,
    annotationLabel,
    annotationCallout,
    annotationCustomType
  } from "d3-svg-annotation";

  export let title = "";

  const activeBars = [];
  //   const activeBars = [];

  export let dataIncoming = [
    { name: "Week vorig jaar", value: 21.5 },
    { name: "Week 1", value: 20 },
    { name: "Week 2", value: 20 },
    { name: "Week 3", value: 20 },
    { name: "Week 4", value: 20 }
  ];

  $: dataSum = sum(data, d => d.value);
  $: dataMax = max(dataIncoming, d => d.value);

  $: dataTestweek = dataIncoming[0];

  const data = dataIncoming.slice(1, 5);

  const padding = { top: 80, right: 10, bottom: 30, left: 50 };

  let width = 800;
  let height = 300;

  function getUrlParam(parameter, defaultvalue) {
    var urlparameter = defaultvalue;
    if (window.location.href.indexOf(parameter) > -1) {
      urlparameter = getUrlVars()[parameter];
    }
    return urlparameter;
  }

  function activeBar(name) {
    return activeBars.includes(name);
  }

  $: xAll = scaleBand()
    .domain(dataIncoming.map(d => d.name))
    .range([padding.left + 50, width - padding.right])
    .padding(0.3);

  $: x = scaleBand()
    .domain(data.map(d => d.name))
    .range([padding.left + 50, width - padding.right])
    .padding(0.3);

  $: y = scaleLinear()
    .domain([0, dataMax])
    .nice(5)
    .range([height - padding.bottom, padding.top]);

  $: yTicks = y.ticks(5);

  $: barWidth = x.bandwidth();

  const type = annotationCustomType(annotationLabel, {
    className: "custom",
    // connector: { type: "curve", end: "arrow" }
    connector: { end: "arrow" }

    // note: { align: "dynamic" }
  });

  const annotations = [
    {
      note: {
        // title: `⚡ ${dataIncoming[0].value} kWh  ⚡`,
        label: `Jullie gaan proberen onder deze lijn te blijven`,
        align: "middle" //cf. right, middle, dynamic
      },
      data: {
        name: "Test week",
        value: dataIncoming[0].value + 1
      },
      x: 300,
      dy: -50,
      dx: 70
    }
    // {
    //   note: {
    //     title: `⚡ ${dataIncoming[1].value} kWh ⚡`,
    //     label: `Week 1`,
    //     align: "middle" //cf. right, middle, dynamic
    //   },
    //   data: {
    //     name: "Week 1",
    //     value: dataIncoming[1].value + 1
    //   },
    //   dy: -60,
    //   dx: 250
    // }
  ];

  const annotationGen = annotation()
    .type(annotationLabel)
    .accessors({
      x: d => x(d.name) + 10,
      y: d => y(d.value)
    })
    .textWrap(150)
    .type(type)
    .annotations(annotations);

  onMount(() => {
    select("#annotations").call(annotationGen);
  });

  //   select("#annotations").call(annotationGen);

  const linegen = line()
    .x(d => xAll(d.name))
    .y(d => y(d.value));
</script>

<style>
  svg {
    position: relative;
    width: 100%;
    height: 300px;
  }
</style>

<h2>{title}</h2>

<div class="chart" bind:clientWidth={width} bind:clientHeight={height}>

  <svg>

    <defs>
      <marker
        id="arrow"
        markerWidth="10"
        markerHeight="10"
        refX="0"
        refY="3"
        orient="auto"
        markerUnits="strokeWidth">
        <path d="M0,0 L0,6 L9,3 z" fill="#f00" />
      </marker>
    </defs>

    <!-- <g class="annotations-manual">

      <path d={linegen(dataIncoming)} stroke="black" fill="none" />

      <path
        d={linegen([
          { name: 'Vorig jaar', value: '39.5' },
          { name: 'Week 1', value: '27.6' }
        ])}
        stroke="black"
        fill="none" />

      <line
        x1="0"
        y1="0"
        x2="200"
        y2="200"
        style="stroke:rgb(255,0,0);stroke-width:2" />
      <text />

      <g class="annotation-note-content" transform="translate({width / 2}, 10)">
        <rect
          class="annotation-note-bg"
          width="88.78125"
          height="33.8328125"
          x="-44.390625"
          y="0"
          fill="white"
          fill-opacity="0" />
        <text
          class="annotation-note-label"
          dx="0"
          y="19.817187500000003"
          fill="grey">
          <tspan x="0" dy="0.8em">Week 1</tspan>
        </text>
        <text class="annotation-note-title" fill="grey" font-weight="bold">
          <tspan x="0" dy="0.8em">⚡ 39.6 kWh ⚡</tspan>
        </text>
      </g>
    </g> -->

    <!-- y axis -->
    <g class="axis y-axis" transform="translate(0,{padding.top})">
      <g
        class="tick"
        transform="translate(0, {y(dataTestweek.value) - padding.top})">
        <line x2="100%" />
        <!-- <text y="-4">Gebruik in testweek</text> -->
      </g>
    </g>

    <!-- x axis -->
    <g class="axis x-axis">
      <g class="tick" transform="translate(15,{height})">
        <text x={barWidth / 2.15} y="-10">{dataTestweek.name}</text>
      </g>

      {#each data as dataPoint, i}
        <g class="tick" transform="translate({x(dataPoint.name)},{height})">
          <text x={barWidth / 2.15} y="-10">{dataPoint.name}</text>
        </g>
      {/each}
    </g>

    <g class="bars">
      <rect
        x="15"
        y={y(dataTestweek.value)}
        width={barWidth - 4}
        height={height - padding.bottom - y(dataTestweek.value)}
        class="testweek" />
      <text
        class="testweek"
        x={barWidth / 2 + 13}
        y={y(dataTestweek.value) + 18}>
        {dataTestweek.value}
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
          {activeBar(dataPoint.name) ? dataPoint.value : '?'}
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
<!-- {getUrlParam('data', 'niks te vinden')} -->
