<script>
  import { onMount } from "svelte";
  import { scaleLinear, scaleBand } from "d3-scale";
  import { ticks, max, sum, mean } from "d3-array";
  import { select } from "d3-selection";
  import { format } from "d3-format";
  import { line, curveBasis } from "d3-shape";

  // Helpers
  const int = format(",.0f");

  // URL paramater parsing and settings
  const params = new URLSearchParams(window.location.search);
  const urlData = params
    .get("data")
    .split(",")
    .map(Number);

  const urlWeeks =
    params.get("weeks") != null ? params.get("weeks").split(",") : "";

  const testWeekString = params.get("testweek");

  const activeBars = urlWeeks;
  const urlWeekNumber = urlData.length - 1;

  // Data
  const barNamesLeft = [
    "Week gemiddelde sep 2018",
    "Week gemiddelde challenge"
  ];

  const barNamesRight = ["Week 1", "Week 2", "Week 3", "Week 4"];
  const testAmount = urlData[0];
  const defaultAmount = testAmount * 0.9;

  // Construct test data and mean data. Show 90% of testdata if no week data is given
  $: dataLeft = [
    {
      name: barNamesLeft[0],
      value: testAmount
    },
    {
      name: barNamesLeft[1],
      value: urlWeekNumber > 0 ? dataRightMean : defaultAmount
    }
  ];

  // Construct week data and mean data. Show 90% of testdata for
  $: dataRight = barNamesRight.map((el, i) => ({
    name: el,
    value: i + 1 <= urlWeekNumber ? urlData[i + 1] : defaultAmount
  }));

  // Data variables
  $: dataSum = sum(dataLeft, d => d.value);
  $: dataMax = max(dataRight, d => d.value); // this has to be the max of all data, also left
  $: dataRightMean = mean(urlData.slice(1), d => d);
  $: dataRightMeanPercentageTestAmount = 1 - testAmount / dataRightMean;

  // Scales
  $: x = scaleBand()
    .domain(dataRight.map(d => d.name))
    .range([width / 2 + padding.left, width - padding.right]) // Offset of 60 pixels, manual first bar with more padding
    .padding(0.25);

  // Scales
  $: x2 = scaleBand()
    .domain(dataLeft.map(d => d.name))
    .range([padding.left * 2, width / 2]) // Offset of 60 pixels, manual first bar with more padding
    .padding(0.35);

  $: y = scaleLinear()
    .domain([0, dataMax])
    // .nice(5)
    .range([height - padding.bottom - 25, padding.top]); // Bars have a min height of 25px, for labels

  // $: yTicks = y.ticks(5);
  $: barWidthLeft = x2.bandwidth();
  $: barWidthRight = x.bandwidth();

  // Settings
  const padding = { top: 240, right: 15, bottom: 120, left: 0 };
  let width = 600;
  let height = 600;
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
        d="M 100 {y(dataLeft[0].value) - 5} C 40 100 100 110 100 110"
        class="arrow" />
      <g transform="translate(100,50)">
        <path d="M -9 67 c 2-3 6-4 9-7-3-1-9-3-11-6" class="arrow" />
      </g>
    </g>

    <!-- arrow for week 1 -->
    <path
      d="M {x(dataRight[0].name) + barWidthRight / 2}
      {y(dataRight[0].value) - 5} C 270 200 270 270 290 170"
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
      <text class="annotation-icon">
        <tspan x="-20" dy="-5">⚡</tspan>
      </text>
      <text class="annotation-icon-status">
        <tspan x="-3" dy="-22">❌</tspan>
      </text>
      <text class="annotation-icon-status">
        <tspan x="-3" dy="-22">✅</tspan>
      </text>
      <text class="annotation-note-title" dx="0" y="30">
        <tspan x="0" dy="-20">Mooi op schema. Ga zo door!</tspan>
      </text>
      <text class="annotation-note-label" dx="0" y="30">
        <tspan x="0" dy="20">
          Jullie zitten nu 7% onder jullie gebruik van vorig jaar.
        </tspan>
      </text>
      <text class="annotation-note-label" dx="0" y="30">
        <tspan x="0" dy="40">In Week 2 gebruikten jullie 5% minder.</tspan>
      </text>
    </g>

    <!-- bars -->
    <g class="bars">
      {#each dataLeft as dataPoint, i}
        <rect
          x={x2(dataPoint.name)}
          y={y(dataPoint.value)}
          width={barWidthLeft - 4}
          height={height - padding.bottom - y(dataPoint.value)}
          class={i == 0 ? 'testweek' : urlWeekNumber == 0 ? 'unknown' : i == 0 ? 'testweek' : dataRightMean <= testAmount ? 'meangood' : 'meanbad'} />
        <text
          class={urlWeekNumber == 0 ? 'unknown' : i == 0 ? 'testweek' : dataRightMean <= testAmount ? 'meangood' : 'meanbad'}
          x={x2(dataPoint.name) + barWidthLeft / 2 - 5}
          y={y(dataPoint.value) + 35}>
          {urlWeekNumber > 0 || i == 0 ? `${int(dataPoint.value)} kWh` : '?'}
        </text>
      {/each}
      {#each dataRight as dataPoint, i}
        <rect
          x={x(dataPoint.name)}
          y={y(dataPoint.value)}
          width={barWidthRight - 4}
          height={height - padding.bottom - y(dataPoint.value)}
          class={urlWeekNumber <= i ? 'unknown' : dataPoint.value < testAmount ? 'weekgood' : 'weekbad'} />
        <text
          class={urlWeekNumber <= i ? 'unknown' : dataPoint.value < testAmount ? 'weekgood' : 'weekbad'}
          x={x(dataPoint.name) + barWidthRight / 2 - 3}
          y={y(dataPoint.value) + 35}>
          {urlWeekNumber > 0 ? `${int(dataPoint.value)} kWh` : '?'}
        </text>
      {/each}
    </g>

    <!-- x axis bar labels -->
    <!-- TODO: just use html for all labels, like we do on dataLeft -->
    <g class="axis x-axis">
      {#each dataRight as dataPoint, i}
        <g class="tick" transform="translate({x(dataPoint.name)},{height})">
          <text x={barWidthRight / 2.15} y={-padding.bottom + 25}>
            {dataPoint.name}
          </text>
        </g>
      {/each}
    </g>
    <!-- y axis line -->
    <g class="axis y-axis" transform="translate(0,{padding.top})">
      <g
        class="tick"
        transform="translate(0, {y(testAmount) - padding.top + 1})">
        <line x1={x2(dataLeft[0].name) + barWidthLeft} x2={width} />
      </g>
    </g>
  </svg>

<!-- x axis bar html labels -->
{#each dataLeft as d}
    <div
      style="position: absolute; top: {height - padding.bottom + 30}px; left: {x2(d.name)}px;
      ; width: 100px ; text-align: center">
      {d.name}
    </div>
  {/each}