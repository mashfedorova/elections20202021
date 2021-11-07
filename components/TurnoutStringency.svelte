<script>
  import { scaleLinear, extent, scaleOrdinal, line } from 'd3';
  import { regressionLinear} from 'd3-regression'
  import { fade } from 'svelte/transition';
  import { interpolatePath } from "d3-interpolate-path";
  import { tweened } from "svelte/motion";
  import  {_} from 'lodash';
  export let data;
  export let width;
  // export let y = 'yTurnout';
  // export let yTick = 'turnoutScaled';
  export let yTickLabel = 'turnout_reg_votes';
  // export let regY = 'turnout_reg_votes';
  export let y = 'turnout_reg_votes';
  export let yHorisontal = 60;
  export let turnoutLabel = 'turnout';
  export let rectTurnoutStringency = '';
  export let regionsHighlight = '';

  // const continents = ["Europe","Middle East","Africa","Americas","Asia-Pacific"];

  const margin= { top:25, right:200, bottom:60, left:200 };
  const backgroundColor = "#f0e8e5";

  $: height = width/2;

  $: extentStringency = extent(data, d => d.stringency_index);
  // $: extentTurnout = extent(data, d => d.turnout_reg_votes);
  // $: extentTurnoutDiff = extent(data, d => +d.turnoutDiff);
  $: extentY = extent(data, y === 'turnout_reg_votes' ? d => d[y] : d => +d[y]);

  $: xScaleStringency= scaleLinear()
    .domain(extentStringency)
    .range([margin.left, width - margin.right]);

  // $: yScaleTurnout = scaleLinear()
  //   .domain(extentTurnout)
  //   .range([height-margin.bottom, margin.top]);

  // $: yScaleTurnoutDiff = scaleLinear()
  // .domain(extentTurnoutDiff)
  // .range([height-margin.bottom, margin.top]);

  $: yScale = scaleLinear()
  .domain(extentY)
  .range([height-margin.bottom, margin.top]);

  // $: colorScale = scaleOrdinal()
  // .domain(continents)
  // .range(['#89848a', '#89848a', '#22ab83', '#89848a', '#89848a'])

  $: stringencyRange = _.range(10, 100, 10);
  // $: turnoutRange = y === 'yTurnout' ? _.range(20, 110, 10) : _.range(-25, 20, 5);

  $: yTurnout = _.range(20, 110, 10);
  $: yTurnoutDiff =  _.range(-25, 20, 5)

  $: yTicksData = yTurnout.map((x,i) => {
    return {turnout_reg_votes:x, turnoutDiff:yTurnoutDiff[i]}
  })

  $: yTicks = yTicksData.map((d) => {
    return {
      ...d,
      yScaled: yScale(d[y])
    }
  })

  $: dataCalc = data.map(d => {
  return {
    x: xScaleStringency(d.stringency_index),
    yScaled: yScale(d[y]),
    d: d.d,
    d1: d.d1,
    d4: d.d4,
    // regionColor: colorScale(d.continent),
    country: d.country,
    continent: d.continent

  };
  });

  //regression line

  const regLinePath = tweened(null, {
    duration: 500,
    interpolate: interpolatePath
  });

  $: reg = regressionLinear()
    .x(d => d.stringency_index)
    .y(d => d[y])

  $: lineReg = line()
    .x(d => xScaleStringency(d[0]))
    .y(d => yScale(d[1]))

  $: regressionData = reg(data)
  // $: regLinePath = lineReg(regressionData)
  $: regLinePath.set(lineReg(regressionData));

  // Horizontal line

  const horizLinePathData = tweened(null, {
  duration: 500,
  interpolate: interpolatePath
  });


  $: horizX = [`${margin.left*0.8}`, `${width-(margin.left*0.8)}`];
  $: horizY = [`${yScale(yHorisontal)+11}`,`${yScale(yHorisontal)+11}`];

  $: horisontalLineData = [{x:horizX[0], y: horizY[0]}, {x:horizX[1], y: horizY[1]}];

  // $: horisontalLineData = horizX.map((x,i) => {
  //   return {x: horizX, y: horizY}
  // })

  $: horizLinePath = line()
    .x(d => d.x)
    .y(d => d.y)

  $: horizLinePathData.set(horizLinePath(horisontalLineData))

  // $: console.log(data)

</script>
<svg width={width} height={height}>
  {#each dataCalc as d}
    <g
    class="animate"
    transform="translate({d.x}, {d.yScaled})"
    style="--x: {d.x}px; --y: {d.yScaled}px;"
    >
    <g transform="scale(0.7)">
      <path
      d={d.d}
      stroke='#000'
      fill= {backgroundColor}
      stroke-width='2'></path>
      <path
      d={d.d1}
      stroke='#000'
      fill= {backgroundColor}
      stroke-width='2'></path>
      <path
      d={d.d4}
      stroke='#000'
      fill= {backgroundColor}
      stroke-width='2'></path>
      <circle
      cx={d.d4 ? 9 : 8}
      cy=23
      r= {d.d4 ? 2 : 3}
      stroke='#000'
      fill= {backgroundColor}
      stroke-width='2'
      ></circle>
    </g>
    </g>
  {/each}

  {#if regionsHighlight}
  {#each dataCalc.filter(d => d.continent === 'Africa') as d}
    <g transform="translate({d.x}, {d.yScaled}) scale(0.7)">
      <rect transition:fade
      x="3"
      y="5"
      width="18"
      height="23"
      fill="{regionsHighlight && d.continent === "Africa" ? '#22ab83': backgroundColor}"
      opacity="0.8"
      filter='url(#highlight)'>
      </rect>
    </g>
  {/each}
  {/if}

  {#each stringencyRange as tick }
    <g class="tick-x" transform="translate({xScaleStringency(tick)}, {height-20})">
      <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
    {/each}
    <!-- svelte-ignore component-name-lowercase -->
    <line
      x1="{xScaleStringency(50)+8}"
      x2="{xScaleStringency(50)+8}"
      y1="{20}"
      y2="{height-40}"
      stroke="#414141"
      stroke-width="1"
      opacity="0.4">
      >
    </line>

  {#each yTicks as tick }
    <!-- <g class="tick-y" transform="translate({margin.left*0.6}, {tick[yTick]+11})" > -->
      <text class="tick-y" x="{margin.left*0.6}" y="{tick.yScaled+11}" fill='#000' opacity='0.7'>
        {tick[yTickLabel]}
      </text>
    <!-- </g>  in:fade="{{delay: 100}}"-->
  {/each}
  <!-- svelte-ignore component-name-lowercase -->
  <!-- <line
    x1="{margin.left*0.8}"
    x2="{width-(margin.left*0.8)}"
    y1="{yScale(yHorisontal)+11}"
    y2="{yScale(yHorisontal)+11}"
    stroke="#414141"
    stroke-width="0.8"
    opacity="0.4"
    >
  </line> -->
  <path
  d="{$regLinePath}"
  stroke='#709afa'
  opacity='0.5'
  fill='none'
  stroke-width='2'
  stroke-dasharray="5,5"
  >
  </path>
  <path
  d="{$horizLinePathData}"
  stroke='#414141'
  fill='none'
  stroke-width='1'
  opacity='0.4'
  >
  </path>
  <text x="{margin.left/1.7}" y="{12}" fill='#000' opacity='0.7'>{turnoutLabel}</text>
  <text x="{width-(margin.left*1.3)}" y="{height-5}" fill='#000' opacity='0.7'>stringency index</text>
  {#if rectTurnoutStringency}
    <g transition:fade>
      <rect x="{xScaleStringency(51)}" y="{yScale(-1)}" width="{width-160-xScaleStringency(51)}" height="{height-yScale(-4)}" fill="#709afa" opacity="0.1"></rect>
      <text x="{width-(margin.left*3)}" y="{height-100}" fill='#709afa' opacity='0.7'>High stringency measures and lower turnout</text>
      <text x="{width-(margin.left*3)}" y="{height-80}" fill='#709afa' opacity='0.7'>in comparison to the previous election</text>
    </g>
  {/if}

  <filter id="highlight">
    <feGaussianBlur stdDeviation="7 7"/>
  </filter>


</svg>

<style>
.animate {
  transition: transform 0.6s ease-out;
  transform: translate(var(--x), var(--y));
}

</style>
