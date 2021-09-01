<script>
import { csv, json, timeParse} from 'd3';
import { onMount } from 'svelte';
import IntroChart from '../components/IntroChart.svelte'

let width = 0;

let data = [];
const parseMonthYear = timeParse("%B %Y");

onMount(async () => {
  const dataRaw = await csv('data/dataFinal.csv', d => {
    return {
    ...d,
    position: +d.position,
    heldMonthYearDate: parseMonthYear(d.heldMonthYear),
    postponedMonthYearDate: parseMonthYear(d.postponedMonthYear),
    }
  })
  data = dataRaw;
  // console.log(data)
});
</script>
<div class="intro-chart" bind:clientWidth={width} >
  <IntroChart {data} width={width}/>
</div>

<style>
.intro-chart {
  margin: 0 auto;
  max-width: 1200px;
}
</style>