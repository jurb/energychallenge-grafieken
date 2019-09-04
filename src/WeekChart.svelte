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
  const barNamesRight = ["Week 1", "Week 2", "Week 3", "Week 4"];
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
  $: dataMax = max(
    [max(dataRight, d => d.value), max(dataLeft, d => d.value)],
    d => d
  ); // this has to be the max of all data, also left
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
    .range([height - padding.bottom - 30, padding.top]); // Bars have a min height of 25px, for labels

  // $: yTicks = y.ticks(5);
  $: barWidthLeft = x2.bandwidth();
  $: barWidthRight = x.bandwidth();

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

      <!-- arrow for testweek -->
      {#if urlWeekNumber == 0}
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
      {/if}

      {#if urlWeekNumber >= 1}
        <path
          d="M {x2(dataLeft[1].name) + barWidthLeft / 2 - 3}
          {y(dataLeft[1].value) - 5} C 210 210 210 210 {x2(dataLeft[1].name) + barWidthLeft / 2}
          170"
          class="arrow" />
        <g
          transform="translate(270.6,{y(dataLeft[1].value) + 0}) rotate(95,0,0)">
          <path d="M -9 67 c 2-3 6-4 9-7-3-1-9-3-11-6" class="arrow" />
        </g>
      {/if}

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

        <!-- Week 0 logic -->
        {#if urlWeekNumber == 0}
          <g id="feedback" class="active">
            <text class="annotation-note-title" dx="0" y="10">
              Lukt het jullie om minder {kind} te verbruiken?
            </text>
            <text class="annotation-note-label" dx="0" y="40">
              <tspan>
                Jullie doel is om aan het eind van de challenge een lager
              </tspan>
            </text>
            <text class="annotation-note-label" dx="0" y="40">
              <tspan y="60" x="0">
                gemiddeld weekverbruik te hebben dan
                {#if kind == 'gas'}
                  {nlformat(testAmount)} m
                  <tspan baseline-shift="super" font-size="10" dx="-3">3</tspan>
                {/if}
                {#if kind == 'stroom'}{nlformat(testAmount)} kWh{/if}
                <tspan>.</tspan>
              </tspan>
            </text>
          </g>
        {/if}

        <!-- Title when things are going bad -->
        {#if dataRightMean >= testAmount}
          <g id="feedback" class="active">
            <text class="annotation-icon-status" x="-5" dy="-10">❌</text>
            <text class="annotation-note-title" dx="0" y="10">
              {#if urlWeekNumber == 4}
                <tspan class="heavy">Eindstand:</tspan>
                helaas, jullie hebben het doel niet gehaald
              {/if}
              {#if urlWeekNumber < 4}
                <tspan class="heavy">Tussenstand:</tspan>
                probeer minder {kind} te gebruiken!
              {/if}
            </text>
            <!-- When things are going meh -->
            {#if parseInt(dataRightMeanPercentageTestAmount * 100) == 0}
              <text class="annotation-note-label" dx="0" y="40">
                <tspan>
                  Na de {counts[parseInt(urlWeekNumber - 1)]} week verbruiken
                  jullie per week
                </tspan>
                <tspan y="60" x="0">
                  gemiddeld net zoveel {kind} als jullie doel van
                  {#if kind == 'gas'}
                    {nlformat(testAmount)} m
                    <tspan baseline-shift="super" font-size="10" dx="-3">
                      3
                    </tspan>
                  {/if}
                  {#if kind == 'stroom'}{nlformat(testAmount)} kWh{/if}
                </tspan>
                <tspan>.</tspan>
              </text>
              <!-- When things are going bad -->
            {:else if dataRightMean >= testAmount}
              <text class="annotation-note-label" dx="0" y="40">
                <tspan>
                  Na de {counts[parseInt(urlWeekNumber - 1)]} week is jullie
                  weekgemiddelde
                  {#if kind == 'gas'}
                    {nlformat(dataRightMean)}m
                    <tspan baseline-shift="super" font-size="10" dx="-3">
                      3
                    </tspan>
                    <tspan dx="-3">.</tspan>
                  {/if}
                  {#if kind == 'stroom'}{nlformat(dataRightMean)} kWh.{/if}
                </tspan>
                <tspan y="60" x="0">
                  <tspan>
                    <tspan dx="-5" />
                    Dat is
                    <tspan class="heavy">
                      {int(dataRightMeanPercentageTestAmount * 100)}%
                    </tspan>
                    &nbsp;boven jullie doel
                    <tspan dx="-3">.</tspan>
                  </tspan>
                </tspan>
              </text>
            {/if}
          </g>
        {/if}

        <!-- When things are going gooood -->
        {#if dataRightMean < testAmount}
          <g id="feedback" class="active">
            <text
              class="annotation-icon-status"
              x="-3"
              dy="-13"
              style="font-size: 1.1em">
              ✅
            </text>
            <text class="annotation-note-title" dx="0" y="10">

              {#if urlWeekNumber == 4}
                <tspan class="heavy">Eindstand:</tspan>
                jullie hebben minder {kind} verbruikt! 🎉
              {/if}
              {#if urlWeekNumber < 4}
                <tspan class="heavy">Tussenstand:</tspan>
                zo gaat ie goed!
              {/if}

            </text>
            <text class="annotation-note-label" dx="0" y="40">
              <tspan>
                Na de {counts[parseInt(urlWeekNumber - 1)]} week is jullie
                weekgemiddelde
                {#if kind == 'gas'}
                  {nlformat(dataRightMean)}m
                  <tspan baseline-shift="super" font-size="10" dx="-3">3</tspan>
                {/if}
                {#if kind == 'stroom'}{nlformat(dataRightMean)} kWh{/if}
                <tspan dx="-4">.</tspan>
              </tspan>
              <tspan y="60" x="0">

                <tspan>
                  <tspan dx="-5" />
                  Dat is
                  <tspan class="heavy">
                    {nlformat(-dataRightMeanPercentageTestAmount * 100)}%
                  </tspan>
                  <tspan dx="-3">
                    &nbsp;beter dan jullie doel, goed gedaan!
                  </tspan>
                </tspan>

              </tspan>
            </text>
          </g>
        {/if}

      </g>

    </g>
    <!-- bars -->
    <g class="bars">
      {#each dataLeft as dataPoint, i}
        <rect
          x={x2(dataPoint.name)}
          y={y(dataPoint.value)}
          width={barWidthLeft - 4}
          height={height - padding.bottom - y(dataPoint.value)}
          class={i == 0 ? 'testweek' : urlWeekNumber == 0 ? 'unknown' : i == 0 ? 'testweek' : dataRightMean >= testAmount ? 'meanbad' : 'meangood'} />
        <text
          class={i == 0 ? 'testweek barleft' : urlWeekNumber == 0 ? 'unknown' : dataRightMean >= testAmount ? 'meanbad barleft' : 'meangood barleft'}
          x={x2(dataPoint.name) + barWidthLeft / 2 - 5}
          y={y(dataPoint.value) + 25}>

          {#if urlWeekNumber == 0 && i == 1}
            ?
          {:else if kind == 'gas'}
            {nlformat(dataPoint.value)} m
            <tspan baseline-shift="super" font-size="8" dx="-2">3</tspan>
          {:else if kind == 'stroom'}{nlformat(dataPoint.value)} kWh{/if}
        </text>
      {/each}
      {#each dataRight as dataPoint, i}
        <rect
          x={x(dataPoint.name)}
          y={y(dataPoint.value)}
          width={barWidthRight - 4}
          height={height - padding.bottom - y(dataPoint.value)}
          class={urlWeekNumber <= i ? 'unknown' : dataPoint.value >= testAmount ? 'weekbad' : 'weekgood'} />
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
  {#each dataLeft as d, i}
    <div
      style="position: absolute; top: {height - padding.bottom + 30}px; left: {x2(d.name) - 4}px;
      ; width: 105px ; text-align: center"
      class:challenge-bar={i == 1}>
      {d.name}
      <div style="font-size: .75em">
        {#if i == 0 && comparisonWeekShort == 'model'}
          vergelijkbaar huishouden
        {:else if i == 0 && comparisonWeekShort}{comparisonWeekShort}{/if}
      </div>
    </div>
  {/each}
</div>

<h3>{gezin}</h3>
