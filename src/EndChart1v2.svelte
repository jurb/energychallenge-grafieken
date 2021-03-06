<script>
  import { scaleLinear, scaleBand } from "d3-scale";
  import { ticks, max, sum, mean, extent, range } from "d3-array";
  import { select } from "d3-selection";
  import { format } from "d3-format";
  import { line, curveBasis } from "d3-shape";
  import { bespaarData } from "./bespaarData.js";

  // props
  export let gezin;
  export let kind;

  // Helpers
  const int = format(",.0f");
  const percentFormat = n =>
    Number(parseFloat(n).toFixed(3)).toLocaleString("nl", { style: "percent" });

  // URL paramater parsing and settings

  const icon = kind == "stroom" ? "⚡️" : "🔥";
  const unit = kind == "stroom" ? "kWh" : "m3";

  // Data

  const bespaarDataSorted = sortBy =>
    [...bespaarData].sort((a, b) => a[sortBy] - b[sortBy]);

  const besparingTicks = [-1, -0.8, -0.6, -0.4, -0.2, 0, 0.2, 0.4, 0.6, 0.8];

  // Scales
  $: x = scaleLinear()
    .domain(extent(range(bespaarData.length)))
    .range([padding.left + 50, width - padding.right - 20]);

  $: y = scaleLinear()
    .domain(extent(besparingTicks))
    // .nice(5)
    .range([height - padding.bottom, padding.top]);

  // Settings
  const padding = { top: 220, right: 15, bottom: 120, left: 15 };
  let width = 600;
  let height = 600;
</script>

<style>
  .tick text {
    font-size: 0.6em;
  }
</style>

<div style="width: {width}; height: {height}">

  <svg id="svg">
    <g class="annotations-manual">
      <rect
        class="annotation-note-bg"
        width="400"
        height="110"
        x="100"
        y="60"
        fill="goldenrod"
        fill-opacity="0.2" />

      <!-- annotation content -->
      <g class="annotation-note-content" transform="translate({width / 2}, 86)">
        <!--  Icon block -->
        {#if kind == 'stroom'}
          <text class="annotation-icon" x="-20" dy="5" style="font-size: 3.5em">
            {icon}
          </text>
        {/if}
        {#if kind == 'gas'}
          <text class="annotation-icon" x="-19" dy="-20">{icon}</text>
        {/if}
        <g id="feedback" class="active">
          <text class="annotation-note-title" dx="0" y="20">
            <tspan class="heavy">Vergelijking met andere gezinnen</tspan>
          </text>
          <text class="annotation-note-label" dx="0" y="40">
            {#if bespaarData.find(el => el.gezin === gezin)[kind] < 0}
              Jullie hebben {percentFormat(Math.abs(bespaarData.find(el => el.gezin === gezin)[kind]))}
              minder {kind} verbruikt dan jullie doel.
            {:else}
              Jullie hebben {percentFormat(bespaarData.find(el => el.gezin === gezin)[kind])}
              meer {kind} verbruikt dan jullie doel.
            {/if}

          </text>
          <text class="annotation-note-label" dx="0" y="60">
            Hieronder zie je hoe de andere gezinnen het hebben gedaan.
          </text>
        </g>
      </g>

      <g class="axis y-axis" transform="translate(0,{padding.top})">
        {#each besparingTicks as tick, i}
          <g
            class="tick ytick"
            transform="translate(0, {y(tick) - padding.top + 1})">
            {#if tick != 0}
              <line
                class="dayline"
                x1={padding.left}
                x2={width - padding.right} />
            {/if}
            {#if tick === 0}
              <line
                class="zeroline"
                x1={padding.left}
                x2={width - padding.right} />
            {/if}

            <text x={padding.left + 10} y="-4">
              {#if tick === besparingTicks[besparingTicks.length - 1]}
                {percentFormat(tick + 1)}
                <tspan dx="3">{kind} verbruik ten opzichte van doelmaand</tspan>
              {:else}{percentFormat(tick + 1)}{/if}
            </text>
          </g>
        {/each}
      </g>

      <g class="dots">
        {#each bespaarDataSorted(kind) as dataPoint, i}
          <circle
            r="7"
            cx={x(i)}
            cy={y(dataPoint[kind])}
            z="9"
            fill={dataPoint.gezin === gezin ? 'rgb(93,205,84)' : kind === 'stroom' ? 'rgb(148,213,238)' : 'rgb(231,177,160)'}
            fill-opacity="1"
            stroke="rgba(0,0,0,0.2)"
            stroke-width="1" />
          <!-- <text class="unknown" x={x(i)} y={y(dataPoint['percentage elek'])}>
          {percentFormat(dataPoint['percentage elek'])}
        </text> -->
        {/each}
      </g>
    </g>
  </svg>

  <!-- x axis bar html labels -->

  <!-- <div
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
  </div> -->

</div>

<h3>{gezin}</h3>
{bespaarData.find(el => el.gezin === gezin).gezin}
