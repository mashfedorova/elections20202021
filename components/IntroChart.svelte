<script>
import { scaleTime, scaleLinear, extent, max, timeFormat, timeParse, scaleBand, scaleOrdinal, map} from 'd3';
import { detach_dev } from 'svelte/internal';
import { fade } from 'svelte/transition'
export let data;
export let width = 0;
export let cancelledBallots;
export let monthsRaw;
const marginDem = { top:25, right:250, bottom:40, left:250 };
const modifier = 10;
export let whichY = 'y';
export let whichX = 'x';
export let dataRectsPostponed;
export let regimes;
export let constData;
export let dataPostponedBallots;
export let marginLeft;

$: margin = { top:20, right:100, bottom:20, left:marginLeft };

$: alldates = extent(constData, d => d.heldMonthYearDate)

$: months = monthsRaw.map(date => timeParse("%B %Y")(date))
$: height = width/2;

$: xScaleTicks = scaleTime()
  .domain(extent(months, d => d))
  .range([margin.left, width-margin.right ])

$: xScale = scaleTime()
  .domain(extent(alldates, d => d))
  .range([margin.left, width-margin.right ])

$: yScale = scaleLinear()
  .domain([0, max(data, d => d.position)])
  .range([height-margin.bottom, margin.top])

$: xScaleDem = scaleLinear()
.domain(extent(data, d => d.demIndexScaleX))
.range([marginDem.left, width-marginDem.right ])

$: yScaleDem = scaleLinear()
.domain(extent(data, d => d.demIndexScaleY))
.range([marginDem.top, height-marginDem.bottom,])

$: xScaleTextReg = scaleOrdinal()
.domain(regimes)
.range([marginDem.left, marginDem.left, width/1.73, width/1.73])

$: yScaleTextReg = scaleOrdinal()
.domain(regimes)
.range([margin.top, height/1.5, margin.top, height/1.5])

$: yScalePostponed = scaleBand()
.domain(map(dataRectsPostponed, d => d.country))
.range([height-margin.bottom, margin.top])

$: dataHeld = data.filter( d => d.werePostponed === 'no')
$: dataPostponed = data.filter( d => d.werePostponed === 'yes')

$: dataHeldCalc = dataHeld.map(d => {
return {
  x: xScale(d.heldMonthYearDate),
  y: yScale(d.position),
  d: d.d,
  d1: d.d1,
  d4: d.d4,
  yDem: yScaleDem(d.demIndexScaleY),
  xDem: xScaleDem(d.demIndexScaleX),
  postponed: d.werePostponed,
  xPost1: xScale(d.postponedMonthYearDate),
  yPost: yScalePostponed(d.country),
  country: d.country
};
});

$: dataPostponedCalc = dataPostponed.map(d => {
return {
  x: xScale(d.heldMonthYearDate),
  y: yScale(d.position),
  d: d.d,
  d1: d.d1,
  d4: d.d4,
  yDem: yScaleDem(d.demIndexScaleY),
  xDem: xScaleDem(d.demIndexScaleX),
  postponed: d.werePostponed,
  xHeld: xScale(d.heldMonthYearDate),
  xPost1: xScale(d.postponedMonthYearDate),
  yPost: yScalePostponed(d.country),
  yHeld: yScalePostponed(d.country),
  country: d.country
};
});

$: cancelledBallotsData = cancelledBallots.map(d => {
return {
  x: xScale(d.postponedMonthYearDate),
  y: yScale(d.position),
  d: d.d,
  d1: d.d1,
  d4: d.d4,
  yDem: yScaleDem(d.demIndexScaleY),
  xDem: xScaleDem(d.demIndexScaleX),  country: d.country,
};
});

$: dataPostponedDates = dataPostponedBallots.map(d => {
return {
  d: d.d,
  d1: d.d1,
  d4: d.d4,
  yDem: yScaleDem(d.demIndexScaleY),
  xDem: xScaleDem(d.demIndexScaleX),
  xPost1: xScale(d.postponedMonthYearDate),
  yPost: yScalePostponed(d.country),
  yHeld: yScalePostponed(d.country),
  xHeld: xScale(d.heldMonthYearDate),
  country: d.country,
};
});
// $: console.log(dataPostponedDates)

// $: dataPostponed = dataRectsPostponed.map(d => {
// return {
//   x: xScale(d.heldMonthYearDate),
//   y: yScale(d.position),
//   yDem: yScaleDem(d.demIndexScaleY),
//   xDem: xScaleDem(d.demIndexScaleX),
//   xPost1: xScale(d.postponedMonthYearDate),
//   yPost: yScalePostponed(d.country)
// };
// });

$: console.log(dataPostponedDates)

</script>
<svg width={width} height={height}>
  {#each dataHeldCalc as d}
  <g class="animate"
  transform="translate({d[whichX]}, {d[whichY]})"
  in:fade="{{delay: d[whichX]*2}}"
  style="--x: {d[whichX]}px; --y: {d[whichY]}px;"
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
    <!-- <g > -->
      <circle
      cx={d.d4 ? 9 : 8}
      cy=23
      r= {d.d4 ? 2 : 3}
      stroke='#000'
      fill= '#fff'
      stroke-width='2'
      ></circle>
  <!-- </g> -->
</g>
    {/each}

{#each dataPostponedDates as d}
        <line
        in:fade="{{delay: 1000}}"
        x1={d.xPost1+23}
        x2={d.xHeld}
        y1={d.yPost+15}
        y2={d.yHeld+15}
        stroke='blue'
        stroke-width='5'
        stroke-opacity='0.2'
        ></line>
  {/each}


  {#each dataPostponedCalc as d}
      <g class="animate"
      transform="translate({d[whichX]}, {d[whichY]})"
      in:fade="{{delay: d[whichX]*2}}"
      style="--x: {d[whichX]}px; --y: {d[whichY]}px;"
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
          <circle
          cx={d.d4 ? 9 : 8}
          cy=23
          r= {d.d4 ? 2 : 3}
          stroke='#000'
          fill= '#fff'
          stroke-width='2'
          ></circle>
          <rect
              x={3}
              y={5}
              width="18"
              height="23"
              fill="blue"
              opacity="0.9"
              filter='url(#highlight)'>
          </rect>
      </g>
    {/each}

  {#each dataPostponedDates as d}
      <g class="animate"
      transform="translate({d.xPost1}, {d.yPost})"
      in:fade="{{delay: 500}}"
      style="--x: {d.xPost1}px; --y: {d.yPost}px;"
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
          <circle
          cx={d.d4 ? 9 : 8}
          cy=23
          r= {d.d4 ? 2 : 3}
          stroke='#000'
          fill= '#fff'
          stroke-width='2'
          ></circle>
        </g>
        <text
        in:fade="{{delay: 1000}}"
          x={200}
          y={d.yPost+15}>
          {d.country}
        </text>
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

    {#each cancelledBallotsData as d}
      <g class="animate" style="--x: {d[whichX]}px; --y: {d[whichY]}px;" in:fade="{{duration: 1000}}" out:fade="{{duration: 100}}">
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
        <!-- <g transform="translate({d[whichX]}, {d[whichY]})"> -->
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
        <!-- </g> -->
      </g>
    {/each}



    {#each regimes as d}
      <text class="regimes" x={xScaleTextReg(d)} y={yScaleTextReg(d)} in:fade="{{duration: 500, delay: 900}}">{d}</text>
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
  transition: transform 1s ease-out;
	}

  .regimes {
    letter-spacing: 0.2em;
  }

  rect {
    transition: all 1s;
  }

</style>