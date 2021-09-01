<script>
import { scaleTime, scaleLinear, line, extent, max } from 'd3';
export let data;
const margin = { top:20, right:50, bottom:100, left:50 };
const width = 800;
const height = 600;
$: test = extent(data, d => d.heldMonthYearDate)

$: xScale = scaleTime()
  .domain(extent(data, d => d.heldMonthYearDate))
  .range([margin.left, width-margin.right ])

$: yScale = scaleLinear()
  .domain([1, max(data, d => d.position)])
  .range([height-margin.bottom, margin.top ])

</script>
<svg width={width} height={height}>
  {#each data as d}
    <path
    d={d.d}
    transform="translate({xScale(d.heldAtAll === 'no' ? d.postponedMonthYearDate : d.heldMonthYearDate)}, {yScale(d.position)})"
    stroke='#000'
    fill= '#fff'
    stroke-width='2'
    ></path>
  {/each}
  {#each data as d}
    <path
    d={d.d1}
    transform="translate({xScale(d.heldAtAll === 'no' ? d.postponedMonthYearDate : d.heldMonthYearDate)}, {yScale(d.position)})"
    stroke='#000'
    fill= '#fff'
    stroke-width='2'
    ></path>
  {/each}
  {#each data as d}
  <g transform="translate({xScale(d.heldAtAll === 'no' ? d.postponedMonthYearDate : d.heldMonthYearDate)}, {yScale(d.position)})">
    {#each data as d}
      <circle
      cx={d.d4 ? 9 : 8}
      cy=23
      r= {d.d4 ? 2 : 3}
      stroke='#000'
      fill= '#fff'
      stroke-width='2'
      ></circle>
    {/each}
  </g>
  {/each}

</svg>
<style>

</style>