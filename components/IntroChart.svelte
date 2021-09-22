<script>
import { scaleTime, scaleLinear, extent, max, timeFormat, timeParse, scaleBand, scaleOrdinal, map} from 'd3';
import { detach_dev } from 'svelte/internal';
import { fade } from 'svelte/transition';

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
export let colorBallot;
export let highlighApr;

const backgroundColor = "#f0e8e5";

$: margin = { top:20, right:100, bottom:30, left:marginLeft };

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
.range([margin.top-6, height/1.5, margin.top-6, height/1.5])

$: ScaleColorReg = scaleOrdinal()
.domain(regimes)
.range(['#d9808c', '#709afa', '#22ab83', '#ccb693'])

$: yScalePostponed = scaleBand()
.domain(map(dataRectsPostponed, d => d.country))
.range([margin.top, height-margin.bottom])

$: scaleOpacity = scaleLinear()
.domain(extent(dataRectsPostponed, d => d.heldDiff))
.range([0.1, 1])

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
  country: d.country,
  opacity: highlighApr === 'no' ? '1' : highlighApr === 'yes' && d.heldMonth === 'April' && d.heldYear === '2020'  ? '1' : highlighApr === 'yes' && d.heldMonth !== 'April' && d.heldYear !== '2020' ? '0.2' : d.heldAtAll === 'no' ? '1' : '0.2'
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
  reg: d.demIndexCat,
  country: d.country,
  opacityDiff: scaleOpacity(d.heldDiff),
  opacity: highlighApr === 'no' ? '1' : highlighApr === 'yes' && d.heldMonth === 'April' && d.heldYear === '2020'  ? '1' : highlighApr === 'yes' && d.heldMonth !== 'April' && d.heldYear !== '2020' ? '0.2' : d.heldAtAll === 'no' ? '1' : '0.2'
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
  opacityDiff: scaleOpacity(d.heldDiff),
  reg: d.demIndexCat,
  country: d.country,
};
});

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
    fill= {backgroundColor}
    stroke-width='2'
    opacity={d.opacity}
    ></path>
    <path
    d={d.d1}
    stroke='#000'
    fill= {backgroundColor}
    stroke-width='2'
    opacity={d.opacity}
    ></path>
    <!-- <g > -->
      <circle
      cx={d.d4 ? 9 : 8}
      cy=23
      r= {d.d4 ? 2 : 3}
      stroke='#000'
      fill= {backgroundColor}
      stroke-width='2'
      opacity={d.opacity}
      ></circle>
  <!-- </g> -->
</g>
    {/each}

{#each dataPostponedDates as d}
        <line
        in:fade="{{duration:100, delay: 500}}"
        x1={d.xPost1+23}
        x2={d.xHeld}
        y1={d.yPost+15}
        y2={d.yHeld+15}
        stroke={ScaleColorReg(d.reg)}
        stroke-width='5'
        stroke-opacity={d.opacityDiff}
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
        fill= {backgroundColor}
        stroke-width='2'
        opacity={d.opacity}
        ></path>
        <path
        d={d.d1}
        stroke='#000'
        fill= {backgroundColor}
        stroke-width='2'
        opacity={d.opacity}
        ></path>
          <circle
          cx={d.d4 ? 9 : 8}
          cy=23
          r= {d.d4 ? 2 : 3}
          stroke='#000'
          fill= {backgroundColor}
          stroke-width='2'
          opacity={d.opacity}
          ></circle>
          <rect
              x={3}
              y={5}
              width="18"
              height="23"
              fill={colorBallot === 'no' ? '#709afa' : ScaleColorReg(d.reg)}
              opacity={highlighApr === 'yes' ? 0.5 : colorBallot === 'no' ? '0.9' : d.opacityDiff}
              filter='url(#highlight)'>
          </rect>
      </g>
    {/each}


  {#each dataPostponedDates as d}
      <g class="animate"
      transform="translate({d.xPost1}, {d.yPost})"
      in:fade="{{duration: 100, delay: 500}}"
      style="--x: {d.xPost1}px; --y: {d.yPost}px;"
      >
        <path
        d={d.d}
        stroke='#000'
        fill= {backgroundColor}
        stroke-width='2'
        ></path>
        <path
        d={d.d1}
        stroke='#000'
        fill= {backgroundColor}
        stroke-width='2'
        ></path>
          <circle
          cx={d.d4 ? 9 : 8}
          cy=23
          r= {d.d4 ? 2 : 3}
          stroke='#000'
          fill= {backgroundColor}
          stroke-width='2'
          ></circle>
        </g>
        <text
        in:fade="{{delay: 100}}"
          x={200}
          y={d.yPost+20}>
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
      <g class="tick" transform="translate({xScaleTicks(month)-modifier}, {height-margin.bottom+20})">
        <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{timeFormat("%b %y")(month)}</text>
      </g>
    {/each}
    {/if}

    {#each cancelledBallotsData as d}
      <g class="animate" style="--x: {d[whichX]}px; --y: {d[whichY]}px;" in:fade="{{duration: 500}}" out:fade="{{duration: 100}}">
        <path
        d={d.d}
        stroke='#000'
        fill= {backgroundColor}
        stroke-width='2'
        ></path>
        <path
        d={d.d1}
        stroke='#000'
        fill= {backgroundColor}
        stroke-width='2'
        ></path>
        <!-- <g transform="translate({d[whichX]}, {d[whichY]})"> -->
          <circle
          cx={d.d4 ? 9 : 8}
          cy=23
          r= {d.d4 ? 2 : 3}
          stroke='#000'
          fill= {backgroundColor}
          stroke-width='2'
          ></circle>
          <rect
          x="3"
          y="5"
          width="18"
          height="23"
          fill="#89848a"
          opacity="0.8"
          filter='url(#highlight)'>
          </rect>
        <!-- </g> -->
      </g>
    {/each}

    {#each regimes as d}
      <text class="regimes" x={xScaleTextReg(d)} y={yScaleTextReg(d)} in:fade="{{duration: 200, delay: 500}}">{d}</text>
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
    transition: transform 0.6s ease-out;
	}

  /* .animateHeld {
    transform: translate(var(--x), var(--y));
    transition: transform 2s ease-out;
  } */

  .regimes {
    letter-spacing: 0.2em;
  }

  rect {
    transition: all 1s;
  }

  path, circle, rect {
    transition: all 0.5s;
  }

</style>