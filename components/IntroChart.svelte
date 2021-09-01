<script>
import { scaleTime, scaleLinear, extent, max, timeFormat,timeParse, scaleBand} from 'd3';

export let data;
export let width = 0;

const monthsRaw = ["April 2021","June 2021","November 2020","June 2020","August 2020","October 2020","December 2020","March 2021","July 2020","May 2021","September 2020","February 2021","March 2020","January 2021","April 2020","May 2020"]
const months = monthsRaw.map(date => timeParse("%B %Y")(date))
const margin = { top:20, right:100, bottom:20, left:100 };
const modifier = 10;

$: height = width/2;

$: xScale = scaleTime()
  .domain(extent(data, d => d.heldMonthYearDate))
  .range([margin.left, width-margin.right ])

$: yScale = scaleLinear()
  .domain([0, max(data, d => d.position)])
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
    {#if width<1050}
      {#each months as month}
        <g class="tick-small" transform="translate({xScale(month)}, {height-margin.bottom+15})">
          <text x="0" y="0" fill='#000' opacity='0.7'>{timeFormat("%b")(month)}</text>
        </g>
      {/each}
      {:else}
      {#each months as month }
        <g class="tick" transform="translate({xScale(month)-modifier}, {height-margin.bottom+10})">
          <text x="0" y="0" fill='#000' opacity='0.7'>{timeFormat("%b %y")(month)}</text>
        </g>
      {/each}
      {/if}
  </svg>

  <p>{width}</p>

<style>
.tick-small {
  font-size: 0.8em;
}
</style>