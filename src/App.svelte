<script>
  import { scaleLinear, scaleBand } from "d3-scale";
  import { ticks, max, sum } from "d3-array";
  import { format } from "d3-format";

  export let title = "";

  const activeBars = ["Test week", "Week 1"];

  export let data = [
    { name: "Test week", value: 17.5 },
    { name: "Week 1", value: 14.6 },
    { name: "Week 2", value: 14.4 },
    { name: "Week 3", value: 14 },
    { name: "Week 4", value: 13 }
  ];

  $: dataSum = sum(data, d => d.value);

  const padding = { top: 20, right: 15, bottom: 20, left: 25 };

  let width = 500;
  let height = 200;

  function activeBar(name) {
    return activeBars.includes(name);
  }

  $: x = scaleBand()
    .domain(data.map(d => d.name))
    .range([padding.left, width - padding.right])
    .padding(0.1);

  $: yScaleValues = scaleLinear()
    .domain([0, max(data, d => d.value)])
    .nice(5)
    .range([height - padding.bottom, padding.top]);

  $: yScaleValuesTicks = yScaleValues.ticks(5);

  $: barWidth = x.bandwidth();
</script>

<style>
  h2 {
    font-family: "Avenir", Helvetica, Arial;
    font-size: 1.1em;
    font-weight: 200;
    text-align: left;
    margin-bottom: 5px;
  }

  .chart {
    width: 100%;
    max-width: 500px;
    margin: 0 auto;
  }

  svg {
    position: relative;
    width: 100%;
    height: 200px;
  }

  .tick {
    font-family: Helvetica, Arial;
    font-size: 0.725em;
    font-weight: 200;
  }

  .tick line {
    stroke: #e2e2e2;
    stroke-dasharray: 2;
  }

  .tick text {
    fill: #ccc;
    text-anchor: start;
  }

  .tick.tick-0 line {
    stroke-dasharray: 0;
  }

  .x-axis .tick text {
    text-anchor: middle;
  }

  rect {
    fill: lightgrey;
    stroke: none;
    opacity: 0.65;
  }

  rect.active {
    fill: #a11;
    stroke: none;
    opacity: 0.65;
  }

  .bars text {
    font-family: "Avenir", Helvetica, Arial;
    font-size: 0.725em;
    font-weight: 200;

    text-anchor: middle;
  }
</style>

<h2>{title}</h2>

<div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
  <svg>

    <!-- y axis -->
    <g class="axis y-axis" transform="translate(0,{padding.top})">
      {#each yScaleValuesTicks as tick}
        <g
          class="tick tick-{tick}"
          transform="translate(0, {yScaleValues(tick) - padding.bottom})">
          <line x2="100%" />
          <text y="-4">
            {tick} {tick === 20 ? ' per 1,000 population' : ''}
          </text>
        </g>
      {/each}
    </g>

    <!-- x axis -->
    <g class="axis x-axis">
      {#each data as dataPoint, i}
        <g class="tick" transform="translate({x(dataPoint.name)},{height})">
          <text x={barWidth / 2.15} y="-4">
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
          y={yScaleValues(dataPoint.value)}
          width={barWidth - 4}
          height={height - padding.bottom - yScaleValues(dataPoint.value)}
          class={activeBar(dataPoint.name) ? 'active' : ''} />
        <!-- <text
          x={x1(dataPoint.name) + x1.bandwidth() / 2 - 3}
          y={yScaleValues(dataPoint.value) - 4}>
          {dataPoint.value} ({dataPoint.percentage})
        </text> -->
      {/each}
    </g>
  </svg>
</div>
<!-- <button on:click={() => tweenedData.set(data2)}>data2</button> -->
{activeBar('1990')}
