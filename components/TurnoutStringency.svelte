<script>
  import { scaleLinear, extent, line, select} from 'd3';
  import { regressionLinear} from 'd3-regression'
  import { fade } from 'svelte/transition';
  import { interpolatePath } from "d3-interpolate-path";
  import { tweened } from "svelte/motion";
  import { afterUpdate } from 'svelte/internal';
  import  {_} from 'lodash';
  import dataRaw from '../public/data/dataFinal';
  export let width;
  export let yTickLabel = 'turnout_reg_votes';
  export let y = 'turnout_reg_votes';
  export let yHorisontal = 60;
  export let turnoutLabel = 'turnout';
  export let rectTurnoutStringency = '';
  export let regionsHighlight = '';

  const backgroundColor = "#f0e8e5";

  const tooltip = select("body").append("div")
  .attr("class", "tooltip")
  .style("text-align", "center")
  .style("display", "block")
  .style("position", "absolute")
  .style("visibility", "hidden")
  .style("font-size", "1rem")
  .style("background", backgroundColor)
  .style("padding", "0.1rem 0.5rem")
  .style("color", "grey")
  .style("border-radius", "1px")
  .style("border", "1px solid #000")
  .style("pointer-events", "none")
  .style("z-index", "11")



  const data = _.chain(dataRaw.filter(d => d.turnout_reg_votes && d.stringency_index))
    .map((d) => {
      return {
        stringency_index: +d.stringency_index,
        turnout_reg_votes: d.turnout_reg_votes,
        turnoutDiff: d.turnoutDiff,
        country: d.country,
        d: d.d,
        d1: d.d1,
        d4: d.d4,
        demIndexCat: d.demIndexCat,
        continent: d.continent,
      }
    })
    .sortBy('turnoutDiff')
    .reverse()
    .value();

  const margin = { top:30, right:20, bottom:100, left:20 };

  $: height = width;


  $: extentStringency = extent(data, d => d.stringency_index);
  $: extentY = extent(data, y === 'turnout_reg_votes' ? d => d[y] : d => +d[y]);

  $: xScaleStringency= scaleLinear()
    .domain(extentStringency)
    .range([margin.left, width - margin.right]);


  $: yScale = scaleLinear()
  .domain(extentY)
  .range([height-margin.bottom, margin.top]);

  $: stringencyRange = _.range(20, 100, 10);


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


  $: horizLinePath = line()
    .x(d => d.x)
    .y(d => d.y)

  $: horizLinePathData.set(horizLinePath(horisontalLineData))

///////////////////////////////////////


    afterUpdate(() => {
    let svg = select('#stringencySvg')
    let ballots  = svg.selectAll('.animate')
      .data(dataCalc)

         ballots.on("mouseenter", (event, d) => {
        tooltip
          .style("visibility", "visible")
          .style("left", event.pageX + "px")
          .style("top", event.pageY + "px")
          .html(
            `<div>${d.country}</div>`
          )
          })

          ballots.on("mouseout", (event, d) => {
        tooltip
          .style("visibility", "hidden")
          })
  })



</script>
<svg id="stringencySvg" width={width} height={height}>
  {#if width > 613}
  {#each dataCalc as d}
    <g
    class="animate"
    transform="translate({d.x}, {d.yScaled})"
    style="--x: {d.x}px; --y: {d.yScaled}px;"
    >
    <g transform="scale(0.7)" class="test">
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
  {:else}
    {#each dataCalc as d}
      <rect
      class="africaRect"
      x='{d.x}'
      y='{d.yScaled}'
      width='7'
      height='8'
      fill={regionsHighlight && d.continent === "Africa" ? '#22ab83': backgroundColor}
      stroke='#000'
      ></rect>
  {/each}
  {/if}

  {#if regionsHighlight && width > 613}
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
      y1="20"
      y2="{height-40}"
      stroke="#414141"
      stroke-width="1"
      opacity="0.2">
      >
    </line>

  {#each yTicks as tick }
      <text class="tick-y" x="{margin.left*0.1}" y="{tick.yScaled+11}" fill='#000' opacity='0.7'>
        {tick[yTickLabel]}
      </text>
  {/each}

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
  opacity='0.2'
  >
  </path>
  <text x="{margin.left/1.7}" y="12" fill='#000' opacity='0.7'>{turnoutLabel}</text>
  <text x="{width-(margin.left*5.2)}" y="{height-5}" fill='#000' opacity='0.7'>stringency index</text>
  {#if rectTurnoutStringency && width > 613}
    <g transition:fade>
      <rect x="{xScaleStringency(51)}" y="{yScale(-1)}" width="{width-xScaleStringency(49)}" height="{height-yScale(-4)}" fill="#709afa" opacity="0.1"></rect>
      <text x="{width-(margin.left*15)}" y="{height-150}" fill='#709afa' opacity='0.7'>High stringency measures and lower turnout</text>
      <text x="{width-(margin.left*15)}" y="{height-130}" fill='#709afa' opacity='0.7'>in comparison to the previous election</text>
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

rect {
  transition: all 0.6s ease-out;
}

</style>
