<script>
  import WeekChart from "./WeekChart.svelte";
  import EndChart1 from "./EndChart1.svelte";
  import EndChart2 from "./EndChart2.svelte";
  import EndChart1v2 from "./EndChart1v2.svelte";
  import EndChart1v3 from "./EndChart1v3.svelte";
  import DayChart from "./DayChart.svelte";

  // Data parameters
  const params = new URLSearchParams(window.location.search);

  const urlData = params
    .get("data")
    .split(",")
    .map(Number);

  const urlWeeks =
    params.get("weeks") != null ? params.get("weeks").split(",") : "";

  const comparisonWeekShort =
    params.get("comparisonshort") != null
      ? params.get("comparisonshort")
      : "verg. huishouden";

  const gezin =
    params.get("gezin") != null ? params.get("gezin") : "Gezin niet bekend";

  const kind = params.get("kind") != null ? params.get("kind") : false; // 'stroom' or 'gas'

  const chartType =
    params.get("charttype") != null ? params.get("charttype") : "weeklyupdate"; // 'weeklyupdate', 'piek'

  const date =
    params.get("date") != null ? params.get("date") : "geen datum gegeven"; // 'weeklyupdate', 'piek'
</script>

{#if chartType == 'weeklyupdate'}
  <WeekChart {urlData} {urlWeeks} {comparisonWeekShort} {gezin} {kind} />
{/if}

{#if chartType == 'endchart1'}
  <EndChart1 {urlData} {urlWeeks} {comparisonWeekShort} {gezin} {kind} />
{/if}

{#if chartType == 'endchart1v2'}
  <EndChart1v2 {gezin} {kind} />
{/if}

{#if chartType == 'endchart1v3'}
  <EndChart1v3 {gezin} {kind} />
{/if}

{#if chartType == 'endchart2'}
  <EndChart2 {urlData} {urlWeeks} {comparisonWeekShort} {gezin} {kind} />
{/if}

{#if chartType == 'daychart'}
  <DayChart {urlData} {gezin} {kind} {date} />
{/if}
