<script>
  import WeekChart from "./WeekChart.svelte";

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
</script>

{#if chartType == 'weeklyupdate'}
  <WeekChart {urlData} {urlWeeks} {comparisonWeekShort} {gezin} {kind} />
{/if}
