<script>
  import { scaleLinear, extent, min, max, schemeSet1, line } from 'd3';
  import { regression, regressionLinear} from 'd3-regression'
  import  {_} from 'lodash';
  export let data;
  export let width;

  const margin= { top:25, right:200, bottom:60, left:200 };
  const backgroundColor = "#f0e8e5";

  $: height = width/2;

  $: extentStringency = extent(data, d => d.stringency_index)
  $: extentTurnout = extent(data, d => d.turnout_reg_votes)

  $: stringencyRange = _.range(10, 100, 10)
  $: turnoutRange = _.range(20, 110, 10)

  $: xScaleStringency= scaleLinear()
    .domain(extentStringency)
    .range([margin.left, width - margin.right])

  $: yScaleTurnout = scaleLinear()
    .domain(extentTurnout)
    .range([height-margin.bottom, margin.top])

  $: dataCalc = data.map(d => {
  return {
    x: xScaleStringency(d.stringency_index),
    yTurnout: yScaleTurnout(d.turnout_reg_votes),
    d: d.d,
    d1: d.d1,
    d4: d.d4,
    country: d.country
  };
  });

  //regression line

  $: reg = regressionLinear()
  .x(d => d.stringency_index)
  .y(d => d.turnout_reg_votes)

  $: lineReg = line()
    .x(d => xScaleStringency(d[0]))
    .y(d => yScaleTurnout(d[1]));

  $: regressionData = reg(data)
  $: regLinePath = lineReg(regressionData)

</script>
<svg width={width} height={height}>
  {#each dataCalc as d }
    <g transform="translate({d.x}, {d.yTurnout}) scale(0.7)">
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
    </g>
  {/each}

  {#each stringencyRange as tick }
    <g class="tick" transform="translate({xScaleStringency(tick)}, {height-10})">
      <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
    {/each}
    <!-- svelte-ignore component-name-lowercase -->
    <line
      x1="{xScaleStringency(50)+8}"
      x2="{xScaleStringency(50)+8}"
      y1="{20}"
      y2="{height-30}"
      stroke="#414141"
      stroke-width="0.8"
      opacity="0.4">
      >
    </line>

  {#each turnoutRange as tick }
    <g class="tick" transform="translate({margin.left*0.6}, {yScaleTurnout(tick)+11})">
      <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
  {/each}
  <!-- svelte-ignore component-name-lowercase -->
  <line
    x1="{margin.left*0.8}"
    x2="{width-(margin.left*0.8)}"
    y1="{yScaleTurnout(60)+11}"
    y2="{yScaleTurnout(60)+11}"
    stroke="#414141"
    stroke-width="0.8"
    opacity="0.4"
    >
  </line>
  <path
  d="{regLinePath}"
  stroke='#709afa'
  opacity='0.5'
  fill='none'
  stroke-width='2'
  stroke-dasharray="5,5"
  >
  </path>
</svg>

<style>

</style>
