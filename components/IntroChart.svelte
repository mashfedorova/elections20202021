<script>
import { scaleTime, scaleLinear, extent, max, timeFormat, timeParse, scaleBand} from 'd3';
import { fade } from 'svelte/transition'
export let data;
export let width = 0;
export let cancelledBallots;
const monthsRaw = ["April 2021","June 2021","November 2020","June 2020","August 2020","October 2020","December 2020","March 2021","July 2020","May 2021","September 2020","February 2021","March 2020","January 2021","April 2020","May 2020"]
const months = monthsRaw.map(date => timeParse("%B %Y")(date))
const margin = { top:20, right:100, bottom:20, left:100 };
const modifier = 10;
export let whichY;

$: height = width/2;

$: xScaleTicks = scaleTime()
  .domain(extent(months, d => d))
  .range([margin.left, width-margin.right ])

$: xScale = scaleTime()
  .domain(extent(data, d => d.heldMonthYearDate))
  .range([margin.left, width-margin.right ])

$: yScale = scaleLinear()
  .domain([0, max(data, d => d.position)])
  .range([height-margin.bottom, margin.top])

$: xScale = scaleBand()
.domain(extent(data, d => d.demIndexCat))
.range([margin.left, width-margin.right ])

  $: calcData = data.map(d => {
  return {
    x: xScale(d.heldMonthYearDate),
    y: yScale(d[whichY]),
    d: d.d,
    d1: d.d1,
    d4: d.d4,
    y1: yScale(1),

  };
});

</script>
<svg width={width} height={height}>
  {#each calcData as d}
  <g class="animate"
  transform="translate({d.x}, {d.y})"
  in:fade="{{delay: d.x*2}}"
  style="--x: {d.x}px; --y: {d.y}px;"
  >
    <path
    d={d.d}
    stroke='#000'
    fill= '#fff'
    stroke-width='2'
    ></path>
    <path
    d={d.d1}
    stroke='#000'
    fill= '#fff'
    stroke-width='2'
    ></path>
    <g >
      <circle
      cx={d.d4 ? 9 : 8}
      cy=23
      r= {d.d4 ? 2 : 3}
      stroke='#000'
      fill= '#fff'
      stroke-width='2'
      ></circle>
  </g>
    </g>
    {/each}

    {#if width<1050}
      {#each months as month}
      <g class="tick-small" transform="translate({xScaleTicks(month)}, {height-margin.bottom+15})">
        <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{timeFormat("%b")(month)}</text>
      </g>
    {/each}
      {:else}
    {#each months as month }
      <g class="tick" transform="translate({xScaleTicks(month)-modifier}, {height-margin.bottom+10})">
        <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{timeFormat("%b %y")(month)}</text>
      </g>
    {/each}
    {/if}

    {#each cancelledBallots as d}
      <g in:fade="{{duration: 1000}}">
        <path
        d={d.d}
        transform="translate({xScale(d.postponedMonthYearDate)}, {yScale(d.position)})"
        stroke='#000'
        fill= '#fff'
        stroke-width='2'
        ></path>
        <path
        d={d.d1}
        transform="translate({xScale(d.postponedMonthYearDate)}, {yScale(d.position)})"
        stroke='#000'
        fill= '#fff'
        stroke-width='2'
        ></path>
        <g transform="translate({xScale(d.postponedMonthYearDate)}, {yScale(d.position)})">
          <circle
          cx={d.d4 ? 9 : 8}
          cy=23
          r= {d.d4 ? 2 : 3}
          stroke='#000'
          fill= '#fff'
          stroke-width='2'
          ></circle>
          <rect
          x="3"
          y="5"
          width="18"
          height="23"
          fill="plum"
          opacity="0.9"
          filter='url(#highlight)'>
          </rect>
        </g>
      </g>
    {/each}

      <filter id="highlight">
        <feGaussianBlur stdDeviation="7 7"/>
      </filter>
  </svg>
<style>
.tick-small {
  font-size: 0.8em;
}
svg {
  display: block;
  /* margin: 0 auto; */
}
	.animate {
	transform: translate(var(--x), var(--y));
  transition: transform 1s linear;
	}

</style>