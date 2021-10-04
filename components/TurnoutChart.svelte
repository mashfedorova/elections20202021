<script>
  import { forceSimulation, forceX, forceY, forceCollide, scaleLinear, scaleOrdinal, extent, min, schemeSet1} from 'd3';
  import { _ } from 'lodash';
  export let data;
  export let width;
  export let x = 'turnout';
  export let xTicks;

  const margin= { top:25, right:200, bottom:40, left:200 };
  const continents = ["Europe","Middle East","Africa","Americas","Asia-Pacific"];
  const regimes = ["democracy","unclassified","authoritarian regime","hybrid regime"];

  $: xTicksTurnout = [20,30, 40, 50, 60, 70, 80, 90, 100];
  $: xTicksTurnoutDiff = [min(data, d => +d.turnoutDiff), -15, -5, 0, 5, 15];

  $: height = width/4;

  $: xScaleTurnout = scaleLinear()
  .domain(extent(data, d => d.turnout_reg_votes))
  .range([margin.left, width - margin.right])

  $: xScaleTurnoutDiff = scaleLinear()
  .domain(extent(data, d => +d.turnoutDiff))
  .range([margin.left, width - margin.right])

  // $: colorScale = scaleOrdinal()
  //   .domain([regimes])
  //   .range(schemeSet1)

  $: colorScale = scaleOrdinal()
    .domain(regimes)
    .range(['#99afe0', 'none', '#cf7480', 'none'])

    $: console.log(colorScale('democracy'))

  $: dataCalc = data.map(d => {
    return {
      turnout: xScaleTurnout(d.turnout_reg_votes),
      turnoutDiff: xScaleTurnoutDiff(d.turnoutDiff),
      turnout_reg_votes: d.turnout_reg_votes,
      country: d.country,
      color: colorScale(d.demIndexCat),
      mandatoryVoting: d.mandatoryVoting
    }
  })

  $: simulation = forceSimulation(dataCalc)
  .force('x', forceX().x(d => d[x]).strength(2))
  .force('y', forceY(width/8).strength(0.8))
  .force('collide', forceCollide(5))
  .stop()

  $: _.times(1000, () => {
    simulation.tick()
  })

</script>

<svg width={width} height={height}>
  {#each dataCalc as d}
    <rect width='7' height='8' x={d.x} y={d.y} fill={d.color} stroke='#191919' stroke-width={d.mandatoryVoting ? '2' : '1'}></rect>
    <!-- <text  x={d.x} y={d.y}>{d.country}</text> -->
  {/each}
  {#if xTicks === 'turnout'}
  <g>
    {#each xTicksTurnout as tick }
    <g class="tick" transform="translate({xScaleTurnout(tick)}, {height-margin.bottom})">
      <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
  {/each}
  </g>
  {/if}
  {#if xTicks === 'difference'}
  <g>
    {#each xTicksTurnoutDiff as tick }
    <g class="tick" transform="translate({xScaleTurnoutDiff(tick)}, {height-margin.bottom})">
      <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
  {/each}
  </g>
  {/if}

  <filter id="highlight">
    <feGaussianBlur stdDeviation="7 7"/>
  </filter>

</svg>

<style>

</style>