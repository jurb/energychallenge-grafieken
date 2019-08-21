<script>
  import { onMount } from "svelte";
  import { scaleLinear, scaleBand } from "d3-scale";
  import { ticks, max, sum } from "d3-array";
  import { select } from "d3-selection";
  import { format } from "d3-format";
  import {
    annotation,
    annotationLabel,
    annotationCallout
  } from "d3-svg-annotation";

  export let title = "";

  const activeBars = ["Test week", "Week 1"];

  export let data = [
    { name: "Test week", value: 11.5 },
    { name: "Week 1", value: 14.6 },
    { name: "Week 2", value: 14.4 },
    { name: "Week 3", value: 14 },
    { name: "Week 4", value: 13 }
  ];

  $: dataSum = sum(data, d => d.value);
  $: dataMax = max(data, d => d.value);
  $: dataTestweek = data[0].value;

  const padding = { top: 45, right: 15, bottom: 30, left: 25 };

  let width = 500;
  let height = 200;

  const annotations = [
    {
      note: {
        title: "Gebruik in testweek",
        label: `⚡ ${data[0].value} kWh ⚡`
      },
      data: {
        name: "Test week",
        value: data[0].value
      },
      x: data[0].value + 60,
      dx: 10,
      dy: -50,
      connector: { end: "arrow" }
    }
  ];

  function activeBar(name) {
    return activeBars.includes(name);
  }

  $: x = scaleBand()
    .domain(data.map(d => d.name))
    .range([padding.left, width - padding.right])
    .padding(0.1);

  $: y = scaleLinear()
    .domain([0, dataMax])
    .nice(5)
    .range([height - padding.bottom, padding.top]);

  $: yTicks = y.ticks(5);

  $: barWidth = x.bandwidth();

  const annotationGen = annotation()
    .type(annotationCallout)
    .accessors({
      x: d => x(d.name),
      y: d => y(d.value)
    })
    .textWrap(200)
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
      <g class="tick" transform="translate(0, {y(dataTestweek) - padding.top})">
        <line x2="100%" />
        <!-- <text y="-4">Gebruik in testweek</text> -->
      </g>
    </g>

    <!-- x axis -->
    <g class="axis x-axis">
      {#each data as dataPoint, i}
        <g class="tick" transform="translate({x(dataPoint.name)},{height})">
          <text x={barWidth / 2.15} y="-10">
            <!-- {width > 380 ? dataPoint.name : formatMobile(dataPoint.name)} -->
            {dataPoint.name}
          </text>
        </g>
      {/each}
    </g>

    <g class="bars">
      {#each data as dataPoint, i}
        <rect
          x={x(dataPoint.name)}
          y={y(dataPoint.value)}
          width={barWidth - 4}
          height={height - padding.bottom - y(dataPoint.value)}
          class={activeBar(dataPoint.name) ? 'active' : ''} />
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
{dataTestweek} {y(dataTestweek)}
