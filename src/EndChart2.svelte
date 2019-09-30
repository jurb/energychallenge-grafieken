<script>
  import { scaleLinear, scaleBand } from "d3-scale";
  import { ticks, max, sum, mean } from "d3-array";
  import { select } from "d3-selection";
  import { format } from "d3-format";
  import { line, curveBasis } from "d3-shape";

  // props
  export let urlWeeks;
  export let urlData;
  export let comparisonWeekShort;
  export let gezin;
  export let kind;

  // Helpers
  const int = format(",.0f");
  const nlformat = n => Number(parseFloat(n).toFixed(1)).toLocaleString("nl");

  // URL paramater parsing and settings

  const icon = kind == "stroom" ? "⚡️" : "🔥";
  const unit = kind == "stroom" ? "kWh" : "m3";

  const activeBars = urlWeeks;
  const urlWeekNumber = urlData.length - 1;

  // Data
  const barNamesLeft = ["Gemiddeld weekverbruik", "Week gemiddelde challenge"];
  const barNamesRight = [
    "1-persoons",
    "2-persoons",
    "3-persoons",
    "4-persoons",
    "5-persoons"
  ];
  const huishoudenType = [
    "flat",
    "tussenwoning",
    "hoekwoning",
    "2 onder 1 kap",
    "vrijstaand"
  ];

  const counts = ["eerste", "tweede", "derde", "vierde"];

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
      value: testAmount
    }
  ];

  // Construct week data and mean data. Show 90% of testdata for
  $: dataRight = barNamesRight.map((el, i) => ({
    name: el,
    value: i + 1 <= urlWeekNumber ? urlData[i + 1] : defaultAmount
  }));

  // Data variables
  $: dataSum = sum(dataLeft, d => d.value);
  $: dataMax = max(
    [max(dataRight, d => d.value), max(dataLeft, d => d.value)],
    d => d
  ); // this has to be the max of all data, also left
  $: dataRightMean = mean(urlData.slice(1), d => d);
  $: dataRightMeanPercentageTestAmount = -(1 - dataRightMean / testAmount);

  // Scales
  $: x = scaleBand()
    .domain(dataRight.map(d => d.name))
    .range([220 + padding.left, width - padding.right]) // Offset of 60 pixels, manual first bar with more padding
    .padding(0.25);

  $: xHuishoudenType = scaleBand()
    .domain(huishoudenType.map(d => d))
    .range([220 + padding.left, width - padding.right]) // Offset of 60 pixels, manual first bar with more padding
    .padding(0.25);

  // Scales
  $: x2 = scaleBand()
    .domain(dataLeft.map(d => d.name))
    .range([0, width / 2]) // Offset of 60 pixels, manual first bar with more padding
    .padding(0.35);

  $: y = scaleLinear()
    .domain([0, dataMax])
    // .nice(5)
    .range([height - padding.bottom - 30, padding.top]); // Bars have a min height of 25px, for labels

  // $: yTicks = y.ticks(5);
  $: barWidthLeft = x2.bandwidth();
  $: barWidthRight = x.bandwidth();

  // Settings
  const padding = { top: 240, right: 15, bottom: 150, left: 0 };
  let width = 600;
  let height = 600;
</script>

<style>

</style>

<div style="width: {width}; height: {height}">

  <svg id="svg">

    <!-- annotations -->
    <g class="annotations-manual">

      <!-- arrow for endchart2 -->
      <!-- <g transform="translate(45,0)">
          <path
            d="M 229.5, 247 C 230, 250, 224, 290, 162, 317.5"
            class="arrow-red" />
          <g transform="translate(179,215.7) rotate(-60,0,0)">
            <path d="M -9 67 c 2-3 6-4 9-7-3-1-9-3-11-6" class="arrow-red" />
          </g>
        </g>
        <text class="annotation-note-title" x="165" y="310">Blijf onder</text>
        <text class="annotation-note-title" x="165" y="330">deze lijn!</text> -->
      <g transform="translate(0,60)">
        <path d="M 145 320 C 180 317.1 180 317.1 229 317.1" class="arrow" />
        <g transform="translate(230,257) rotate(0,0,0)">
          <path d="M -9 67 c 2-3 6-4 9-7-3-1-9-3-11-6" class="arrow" />
        </g>
      </g>

      <!-- Annotation section -->
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

        <!-- Title when things are going bad -->
        <g id="feedback" class="active">
          <!-- <text class="annotation-icon-status" x="-5" dy="-10">❌</text> -->
          <text class="annotation-note-title" dx="0" y="10">
            <tspan class="heavy">Vergelijking andere huishoudens</tspan>
          </text>
          <text class="annotation-note-label" dx="0" y="40">
            <tspan>
              Bekijk hier hoe jullie {kind} verbruik zich verhoudt
              <!-- {#if kind == 'gas'}
                  {nlformat(dataRightMean)}m
                  <tspan baseline-shift="super" font-size="10" dx="-3">3</tspan>
                  <tspan dx="-3">.</tspan>
                {/if}
                {#if kind == 'stroom'}{nlformat(dataRightMean)} kWh.{/if} -->
            </tspan>
            <tspan y="60" x="0">
              <tspan>
                <tspan dx="-5" />
                tot andere huishoudens.
                <!-- <tspan class="heavy">
                    {int(dataRightMeanPercentageTestAmount * 100)}%
                  </tspan>
                  &nbsp;boven jullie doel
                  <tspan dx="-3">.</tspan> -->
              </tspan>
            </tspan>
          </text>
        </g>

      </g>

    </g>
    <!-- bars -->
    <g class="bars">
      <rect
        x="60"
        y={y(dataLeft[1].value)}
        width={barWidthLeft - 4}
        height={height - padding.bottom - y(dataLeft[1].value)}
        class="endchart2left" />
      <text
        class="endchart2left barleft"
        x={60 + barWidthLeft / 2 - 5}
        y={y(dataLeft[1].value) + 25}>

        {#if kind == 'gas'}
          {nlformat(dataLeft[1].value)} m
          <tspan baseline-shift="super" font-size="8" dx="-2">3</tspan>
        {:else if kind == 'stroom'}{nlformat(dataLeft[1].value)} kWh{/if}
      </text>
      {#each dataRight as dataPoint, i}
        <rect
          x={x(dataPoint.name)}
          y={y(dataPoint.value)}
          width={barWidthRight - 4}
          height={height - padding.bottom - y(dataPoint.value)}
          class={kind === 'stroom' ? 'endchart2rightstroom' : 'endchart2rightgas'} />
        <text
          class={urlWeekNumber <= i ? 'unknown' : dataPoint.value >= testAmount ? 'weekbad' : 'weekgood'}
          x={x(dataPoint.name) + barWidthRight / 2 - 3}
          y={y(dataPoint.value) + 20}>
          {#if i >= urlWeekNumber}
            ?
          {:else if kind == 'gas'}
            {nlformat(dataPoint.value)} m
            <tspan baseline-shift="super" font-size="8" dx="-2">3</tspan>
          {:else if kind == 'stroom'}{nlformat(dataPoint.value)} kWh{/if}
        </text>
      {/each}
    </g>

    <!-- x axis bar labels -->
    <!-- TODO: just use html for all labels, like we do on dataLeft -->
    {#if kind === 'gas'}
      <g class="axis x-axis">
        {#each huishoudenType as dataPoint, i}
          <g
            class="tick smaller"
            transform="translate({xHuishoudenType(dataPoint)},{height})">
            <text x={barWidthRight / 2.15} y={-padding.bottom + 25}>
              {dataPoint}
            </text>
          </g>
        {/each}
      </g>
    {/if}

    {#if kind === 'stroom'}
      <g class="axis x-axis">
        {#each dataRight as dataPoint, i}
          <g
            class="tick smaller"
            transform="translate({x(dataPoint.name)},{height})">
            <text x={barWidthRight / 2.15} y={-padding.bottom + 25}>
              {dataPoint.name}
            </text>
          </g>
        {/each}
      </g>
    {/if}

    <!-- y axis line -->
    <!-- <g class="axis y-axis" transform="translate(0,{padding.top})">
      <g
        class="tick"
        transform="translate(0, {y(testAmount) - padding.top + 1})">
        <line
          class="weekline"
          x1={x2(dataLeft[0].name) + barWidthLeft}
          x2={width} />
      </g>
    </g> -->
  </svg>

  <!-- x axis bar html labels -->
  <div
    style="position: absolute; top: {height - padding.bottom + 60}px; left: {35}px;
    ; width: 160px ; text-align: center"
    class="challenge-bar">
    Jullie verbruik
    <div style="font-size: .75em">tijdens de challenge</div>
  </div>

  <div
    style="position: absolute; top: {height - padding.bottom + 60}px; left: {290 - 4}px;
    ; width: 260px ; text-align: center"
    class="challenge-bar">
    Verbruik vergelijkbare huishoudens
    {#if kind === 'stroom'}
      <div style="font-size: .75em">in aantal gezinsleden</div>
    {/if}
    {#if kind === 'gas'}
      <div style="font-size: .75em">per type woning</div>
    {/if}
  </div>

</div>

<h3>{gezin}</h3>
{dataRight.map(el => el.value)}
