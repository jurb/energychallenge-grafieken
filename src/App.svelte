<script>
  import { onMount } from "svelte";
  import { scaleLinear, scaleBand } from "d3-scale";
  import { ticks, max, sum } from "d3-array";
  import { select } from "d3-selection";
  import { format } from "d3-format";
  import { curveBasis } from "d3-shape";
  import {
    annotation,
    annotationLabel,
    annotationCallout,
    annotationCustomType
  } from "d3-svg-annotation";

  export let title = "";

  const activeBars = ["Week 1"];

  export let dataIncoming = [
    { name: "Vergelijkingsweek", value: 16.5 },
    { name: "Week 1", value: 13.6 },
    { name: "Week 2", value: 17.4 },
    { name: "Week 3", value: 14 },
    { name: "Week 4", value: 13 }
  ];

  $: dataSum = sum(data, d => d.value);
  $: dataMax = max(dataIncoming, d => d.value);

  $: dataTestweek = dataIncoming[0];

  const data = dataIncoming.slice(1, 5);

  const padding = { top: 45, right: 15, bottom: 30, left: 60 };

  let width = 500;
  let height = 200;

  function activeBar(name) {
    return activeBars.includes(name);
  }

  $: x = scaleBand()
    .domain(data.map(d => d.name))
    .range([padding.left + 50, width - padding.right])
    .padding(0.2);

  $: y = scaleLinear()
    .domain([0, dataMax])
    .nice(5)
    .range([height - padding.bottom, padding.top]);

  $: yTicks = y.ticks(5);

  $: barWidth = x.bandwidth();

  const type = annotationCustomType(annotationLabel, {
    className: "custom"
    // connector: { type: "curve", end: "arrow" }
    // note: { align: "dynamic" }
  });

  const annotations = [
    {
      note: {
        title: `⚡ ${dataIncoming[0].value} kWh  ⚡`,
        label: `Vergelijkingsmaand`,
        align: "middle" //cf. right, middle, dynamic
      },
      data: {
        name: "Test week",
        value: dataIncoming[0].value + 1
      },
      x: 45,
      dy: -20,
      dx: 0
    },
    {
      note: {
        title: `⚡ ${dataIncoming[1].value} kWh ⚡`,
        label: `Week 1`,
        align: "middle" //cf. right, middle, dynamic
      },
      data: {
        name: "Week 1",
        value: dataIncoming[1].value + 1
      },
      dy: -20,
      dx: 0
    }
  ];

  const annotationGen = annotation()
    .type(annotationLabel)
    .accessors({
      x: d => x(d.name) + 30,
      y: d => y(d.value)
    })
    .textWrap(250)
    .type(type)
    .annotations(annotations);

  onMount(() => {
    select("#annotations").call(annotationGen);
  });
</script>

<style>

</style>

<h2>{title}</h2>

<div class="chart" bind:clientWidth={width} bind:clientHeight={height}>

  <svg>

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

      {#each data as dataPoint, i}
        <rect
          x={x(dataPoint.name)}
          y={y(dataPoint.value)}
          width={barWidth - 4}
          height={height - padding.bottom - y(dataPoint.value)}
          class={activeBar(dataPoint.name) ? 'active' : 'inactive'} />
        <!-- <text
          x={x1(dataPoint.name) + x1.bandwidth() / 2 - 3}
          y={y(dataPoint.value) - 4}>
          {dataPoint.value} ({dataPoint.percentage})
        </text> -->
      {/each}
    </g>

    <g id="annotations" />

  </svg>
</div>
<!-- <button on:click={() => tweenedData.set(data2)}>data2</button> -->
<!-- {activeBar('1990')} -->
<!-- {annotationGen()} -->
{dataTestweek} {y(dataTestweek)} {barWidth}
