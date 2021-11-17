<script>
  import { scaleTime, scaleLinear, extent, max, timeFormat, timeParse, scaleBand, scaleOrdinal, map, select, selectAll, transition} from 'd3';
  import { onMount, afterUpdate, beforeUpdate } from 'svelte/internal';
  import { fade } from 'svelte/transition';
  import  { _ } from 'lodash';

  export let data;
  export let cancelledBallots;
  export let monthsRaw;
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
  export let legend;
  export let opacityValue;
  export let highlightLines;
  export let width;

  const tooltip = select("body").append("div")
  .attr("class", "tooltip")
  .style("text-align", "center")
  .style("display", "block")
  .style("position", "absolute")
  .style("visibility", "hidden")
  .style("font-size", "1rem")
  .style('font-family', 'Montaga')
  .style("background", "rgba(255,255,255,0.9)")
  .style("padding", "0.1rem 0.5rem")
  .style("color", "grey")
  .style("border-radius", "5px")
  .style("border", "1px solid grey")
  // .style("transform", "translate(-30%, -50%)")
  .style("pointer-events", "none")
  // .style("cursor", "pointer")
  .style("z-index", "11")


const backgroundColor = "#f0e8e5";
// import testing from "./../public/data/test.js";
// import dataLines from "./../public/data/dataLines.js";

let margin;
$: if (width < 550 && legend === 'no') {
  margin = { top:5, right:20, bottom:30, left:5 };
} else if (width < 920 && legend === 'yes') {
  margin = { top:20, right:30, bottom:30, left:110 };
} else {
  margin = { top:20, right:130, bottom:30, left:marginLeft };
}

let marginDem;
$: if (width < 920) {
  marginDem = { top:25, right:10, bottom:40, left:5 };
} else {
  marginDem = { top:25, right:250, bottom:40, left:250 };
}

$: height = width/2;

// $: console.log(width, margin, marginDem);

$: alldates = extent(constData, d => d.heldMonthYearDate);

$: months = monthsRaw.map(date => timeParse("%B %Y")(date));
const monthsSmall = ["March 2020","December 2020","September 2021"];
const monthsSmallDates = monthsSmall.map(date => timeParse("%B %Y")(date))

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
.range([marginDem.top, height-marginDem.bottom])

$: xScaleTextReg = scaleOrdinal()
.domain(regimes)
.range([marginDem.left, marginDem.left, width/1.73, width/1.73])

$: yScaleTextReg = scaleOrdinal()
.domain(regimes)
.range([margin.top-6, height/1.5, margin.top-6, height/1.5])

$: ScaleColorReg = scaleOrdinal()
.domain(regimes)
.range(['#d68993', '#709afa', '#22ab83', '#ccb693'])

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
  opacity: highlighApr === 'no' ? '1' : highlighApr === 'yes' && d.heldMonth === 'April' && d.heldYear === '2020'  ? '1' :  '0.2',
  opFadeout: '0.2',
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
  reg: d.demIndexCat,
  country: d.country,
  opacityDiff: scaleOpacity(d.heldDiff),
  opacity: highlighApr === 'no' ? '1' : highlighApr === 'yes' && d.heldMonth === 'April' && d.heldYear === '2020'  ? '1' : '0.2',
  opacityLines: highlightLines === 'no' ? '1' : d.country === 'Ethiopia' ? '1' : '0.3',
  opFadeout: '0.2',
  country: d.country,
  demIndexCat: d.demIndexCat

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
  opacityLines: highlightLines === 'no' ? '1' : d.country === 'Ethiopia' ? '1' : '0.3',
  country: d.country,
  demIndexCat: d.demIndexCat

};
});


$: console.log(cancelledBallots, cancelledBallotsData)

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
  opacityLines: highlightLines === 'no' ? '1' : d.country === 'Ethiopia' ? '1' : '0.3'
};
});

$: dataRegime1 = dataPostponedCalc.filter(d => d.country === 'Chile');
$: dataRegime2 = cancelledBallotsData.filter(d => d.country === 'Armenia' || d.country === 'Chad' || d.country === 'Somalia');

// $: console.log(dataHeldCalc)
// const sel = select(this);
// const svg = sel.append('svg').attr('class', 'control');
// const rs = rough.svg(svg.node());
// // const svg = select('#svg');
// // const rc = rough.svg(svg.node())
// // svg.appendChild(rc.circle(80, 120, 50));

// console.log(sel)

//   const svg = document.getElementById('svg');
//   const rs = rough.svg(svg.node());


// onMount(() => {
//   const svg = document.getElementById('svg');
//   const rc = rough.svg(svg);
//   // svg.appendChild(rc.circle(80, 120, 50));

//   // for (const x of dataPostponedDates) { console.log(x); }
//   dataLines.forEach(d => {
//       svg.appendChild(rc.line(d.xPost1, d.yPost, d.xHeld, d.yHeld, {strokeWidth: 5, stroke: 'lightblue'}));
//       // console.log(d.xPost1, d.yPost, d.xPost1, d.yHeld)
//       // svg.appendChild(rc.line(356.1636728366919 ,20 ,356.1636728366919,20));
//       // svg.appendChild(rc.line(356 ,20 ,356,20));
//   // svg.appendChild(rc.circle(80, 120, 50));
//     })

    //   <line
    // in:fade="{{duration:100, delay: 500}}"
    // x1={d.xPost1+23}
    // x2={d.xHeld}
    // y1={d.yPost+15}
    // y2={d.yHeld+15}
    // stroke={ScaleColorReg(d.reg)}
    // stroke-width='5'
    // stroke-opacity={highlightLines === 'no' ? d.opacityDiff : highlightLines === 'yes' && d.country === 'Ethiopia' ? d.opacityDiff : d.opacityLines }
    // opacity={d[opacityValue]}
    // ></line>
  // })

//  $:svg = select('svg')
//  $: ballots = svg.selectAll('.animate')
//  .data(dataHeldCalc)

//   $: console.log(ballots)
//   $: ballots.on("mouseover", (event, d) => {
//         const current = event.currentTarget;
//         console.log(current)
//         tooltip
//           .style("visibility", "visible")
//           .style("top", (d[whichY]) +"px")
//           .style("left", (d[whichX]) + "px")
//           .html(
//             `<div>${d.country}</div>`
//           )
//   })


// $: svg = select('svg');


    afterUpdate(() => {
    let svg = select('svg')
    let ballots  = svg.selectAll('.animate')
      .data(dataHeldCalc)

    let ballotsPost = svg.selectAll('.rectPost')
      .data(dataPostponedCalc)

    let ballotsCancelled = svg.selectAll('.rectCancelled')
      .data(cancelledBallotsData)

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

          ballotsPost.on("mouseenter", (event, d) => {
        tooltip
          .style("visibility", "visible")
          .style("left", event.pageX + "px")
          .style("top", event.pageY + "px")
          .html(
            `<div>${d.country}</div>`
          )
          })

          ballotsPost.on("mouseout", (event, d) => {
        tooltip
          .style("visibility", "hidden")
          })

            ballotsCancelled.on("mouseenter", (event, d) => {
        tooltip
          .style("visibility", "visible")
          .style("left", event.pageX + "px")
          .style("top", event.pageY + "px")
          .html(
            `<div>${d.country}</div>`
          )
        })

          ballotsCancelled.on("mouseout", (event, d) => {
        tooltip
          .style("visibility", "hidden")
          })

  })

// $: ballots.on("mouseover", (event, d) => {
//         const current = event.currentTarget;
//         console.log(current)
//         tooltip
//           .style("visibility", "visible")
//           .style("top", (d[whichY]) +"px")
//           .style("left", (d[whichX]) + "px")
//           .html(
//             `<div>${d.country}</div>`
//           )
//   })

// function handleMousemove() {
//    ballots.on("mouseenter", (event, d) => {
//         tooltip
//           .style("visibility", "visible")
//           .style("left", event.pageX + "px")
//           .style("top", event.pageY + "px")
//           .html(
//             `<div>${d.country}</div>`
//           )
//   })

//   // ballotsPost.on("mouseenter", (event, d) => {
//   //       tooltip
//   //         .style("visibility", "visible")
//   //         .style("left", event.pageX + "px")
//   //         .style("top", event.pageY + "px")
//   //         .html(
//   //           `<div>${d.country}</div>`
//   //         )
//   // })

//   //   ballotsCancelled.on("mouseenter", (event, d) => {
//   //       tooltip
//   //         .style("visibility", "visible")
//   //         .style("left", event.pageX + "px")
//   //         .style("top", event.pageY + "px")
//   //         .html(
//   //           `<div>${d.country}</div>`
//   //         )
//   // })
// }


function handleMouseleave() {
    ballots.on("mouseout", (event,d) => {
    tooltip.style("visibility", "hidden")
  })

  //   ballotsPost.on("mouseout", (event,d) => {
  //   tooltip.style("visibility", "hidden")
  // })

  //   ballotsCancelled.on("mouseout", (event,d) => {
  //   tooltip.style("visibility", "hidden")
  // })
}


  // $:console.log(width, ballots)

</script>

<!-- <svg width={width} height={height} id="svg"   on:mouseenter={handleMousemove}> -->
<svg width={width} height={height} id="svg" >

  {#if width > 920}
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
    opacity={d[opacityValue]}
    ></path>
    <path
    d={d.d1}
    stroke='#000'
    fill= {backgroundColor}
    stroke-width='2'
    opacity={d[opacityValue]}
    ></path>
    <!-- <g > -->
      <circle
      cx={d.d4 ? 9 : 8}
      cy=23
      r= {d.d4 ? 2 : 3}
      stroke='#000'
      fill= {backgroundColor}
      stroke-width='2'
      opacity={d[opacityValue]}
      ></circle>
  <!-- </g> -->
</g>
    {/each}

{#each dataPostponedDates as d}
        <!-- svelte-ignore component-name-lowercase -->
        <line
        in:fade="{{duration:100, delay: 500}}"
        x1={d.xPost1+23}
        x2={d.xHeld}
        y1={d.yPost+15}
        y2={d.yHeld+15}
        stroke={ScaleColorReg(d.reg)}
        stroke-width='5'
        stroke-opacity={highlightLines === 'no' ? d.opacityDiff : highlightLines === 'yes' && d.country === 'Ethiopia' ? d.opacityDiff : d.opacityLines }
        opacity={d[opacityValue]}
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
        opacity={d[opacityValue]}
        ></path>
        <path
        d={d.d1}
        stroke='#000'
        fill= {backgroundColor}
        stroke-width='2'
        opacity={d[opacityValue]}
        ></path>
          <circle
          cx={d.d4 ? 9 : 8}
          cy=23
          r= {d.d4 ? 2 : 3}
          stroke='#000'
          fill= {backgroundColor}
          stroke-width='2'
          opacity={d[opacityValue]}
          ></circle>
          <rect
              class="rectPost"
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
        opacity={d[opacityValue]}
        ></path>
        <path
        d={d.d1}
        stroke='#000'
        fill= {backgroundColor}
        stroke-width='2'
        opacity={d[opacityValue]}
        ></path>
          <circle
          cx={d.d4 ? 9 : 8}
          cy=23
          r= {d.d4 ? 2 : 3}
          stroke='#000'
          fill= {backgroundColor}
          stroke-width='2'
          opacity={d[opacityValue]}
          ></circle>
        </g>
        <text
        in:fade="{{delay: 100}}"
        opacity={d[opacityValue]}
          x={200}
          y={d.yPost+20}>
          {d.country}
        </text>
    {/each}

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
          class="rectCancelled"
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

    {#each dataPostponedDates.filter(d => d.country === 'Syria') as d}
      <text  class="post-label" x={d.xPost1-24} y='{d.yPost-5}'>postponed</text>
    {/each}

    {#each dataPostponedDates.filter(d => d.country === 'Syria') as d}
      <text  class="post-label" x='{d.xHeld-7}' y='{d.yHeld-5}'>held</text>
    {/each}

    {#if regimes === 'yes'}
    {#each dataRegime1 as d}
      <text class="regimes" x='{d[whichX]}' y='{d[whichY]-10}' in:fade="{{duration: 200, delay: 500}}">{d.demIndexCat}</text>
    {/each}

    {#each dataRegime2 as d}
      <text class="regimes" x='{d[whichX]}' y='{d[whichY]-10}' in:fade="{{duration: 200, delay: 500}}">{d.demIndexCat}</text>
    {/each}
    {/if}


    <!-- {#if legend === 'yes'}
      <text class="post-label" transition:fade="{{delay: 600, duration: 100}}" x="26%" y="10"> postponed</text>
      <text class="post-label" transition:fade="{{delay: 600, duration: 100}}" x="41.5%" y="10"> held</text>
    {/if} -->


    {/if}

  <!-- smaller screens -->
  {#if width < 920}
    {#each dataHeldCalc as d}
      <rect
      in:fade="{{delay: d[whichX]*2}}"
      x='{d[whichX]}'
      y='{d[whichY]}'
      width='7'
      height='8'
      fill={backgroundColor}
      stroke='#000'
      opacity={d[opacityValue]}
      ></rect>
    {/each}

{#each dataPostponedDates as d}
        <!-- svelte-ignore component-name-lowercase -->
        <line
        in:fade="{{duration:100, delay: 500}}"
        x1={d.xPost1+2}
        x2={d.xHeld}
        y1={d.yPost+15}
        y2={d.yHeld+15}
        stroke={ScaleColorReg(d.reg)}
        stroke-width='2'
        stroke-opacity={highlightLines === 'no' ? d.opacityDiff : highlightLines === 'yes' && d.country === 'Ethiopia' ? d.opacityDiff : d.opacityLines }
        opacity={d[opacityValue]}
        ></line>
  {/each}

  {#each dataPostponedCalc as d}
      <rect
      in:fade="{{delay: d[whichX]*2}}"
      x='{d[whichX]}'
      y='{legend === 'yes' ? d[whichY]+10 : d[whichY] }'
      width='7'
      height='8'
      fill={colorBallot === 'no' ? '#709afa' : ScaleColorReg(d.reg)}
      stroke='#000'
      opacity={d[opacityValue]}
      ></rect>
  {/each}

  {#each dataPostponedDates as d}
      <!-- <rect
      x='{d.xPost1}'
      y='{d.yPost}'
      width='5'
      height='5'
      fill={colorBallot === 'no' ? '#709afa' : ScaleColorReg(d.reg)}
      opacity={highlighApr === 'yes' ? 0.5 : colorBallot === 'no' ? '0.9' : d.opacityDiff}
      filter='url(#highlight)'
      ></rect> -->
      <text class="country-label-postponed"
      in:fade="{{delay: 100}}"
      opacity={d[opacityValue]}
      x={10}
      y={d.yPost+20}>
      {d.country}
      </text>
    {/each}

    {#each cancelledBallotsData as d}
      <rect
      x='{d[whichX]}'
      y='{d[whichY]}'
      width='7'
      height='8'
      fill="#89848a"
      stroke='#000'
      opacity="0.8"
      ></rect>
    {/each}

      {#each dataPostponedDates as d}
      <rect
      x='{d.xPost1}'
      y='{d.yPost+10}'
      width='7'
      height='8'
      fill={backgroundColor}
      stroke='#000'
      opacity="0.8"
      ></rect>
    {/each}

      {#each dataPostponedDates.filter(d => d.country === 'Syria') as d}
        <text  class="post-label-small" x={d.xPost1-28} y='{d.yPost-5}'>postponed</text>
      {/each}

      {#each dataPostponedDates.filter(d => d.country === 'Syria') as d}
        <text  class="post-label-small" x='{d.xHeld-15}' y='{d.yHeld-5}'>held</text>
      {/each}

    {#if regimes === 'yes'}
    {#each dataRegime1 as d}
      <text class="regimes-small" x='{d[whichX]}' y='{d[whichY]-7}' in:fade="{{duration: 200, delay: 500}}">{d.demIndexCat}</text>
    {/each}
    {#each dataRegime2 as d}
      <text class="regimes-small" x='{d[whichX]}' y='{d[whichY]-10}' in:fade="{{duration: 200, delay: 500}}">{d.demIndexCat}</text>
    {/each}
    {/if}

  {/if}

    {#if width<460}
      {#each monthsSmallDates as month}
      <g class="tick-small" transform="translate({xScaleTicks(month)}, {height-margin.bottom+15})">
        <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{timeFormat("%b")(month)}</text>
      </g>
    {/each}
      {:else if width>459 && width < 1100}
    {#each months as month }
      <g class="tick=small" transform="translate({xScaleTicks(month)-modifier}, {height-margin.bottom+20})">
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

      <defs>
        <filter id="highlight">
          <feGaussianBlur stdDeviation="7 7"/>
        </filter>
      </defs>
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

  path, circle, rect, line, text {
    transition: all 0.5s;
  }

  .post-label {
    letter-spacing: 0.1em;
    fill: rgb(117, 117, 117)
  }

  @media only screen and (max-width: 460px) {
    .country-label-postponed {
      font-size: 12px;
    }

      .post-label-small {
        letter-spacing: 0em;
        fill: rgb(117, 117, 117);
        font-size: 13px;
      }

      .regimes-small {
        letter-spacing: 0em;
      }
  }

</style>