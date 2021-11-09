<script>
  import { forceSimulation, forceX, forceY, forceCollide, scaleLinear, scaleOrdinal, median, mean} from 'd3';
  import { _ } from 'lodash';
  export let data;
  export let width;
  export let x = 'turnout';
  export let xTicks;


  const margin= { top:25, right:200, bottom:40, left:200 };
  const continents = ["Europe","Middle East","Africa","Americas","Asia-Pacific"];
  const regimes = ["democracy","unclassified","authoritarian regime","hybrid regime"];
  const backgroundColor = "#f0e8e5";
  const xTicksTurnout = [20,30, 40, 50, 60, 70, 80, 90, 100];

  // $: medianTurnout = mean(data, d => d.turnout_reg_votes);
  // $: medianTurnoutDiff = mean(data, d => d.turnoutDiff);
  // $: console.log(medianTurnout, medianTurnoutDiff)

  $: xTicksTurnoutDiff = [-25, -15, -5, 0, 5, 15];

  $: height = width/4;

  $: xScaleTurnout = scaleLinear()
  // .domain(extent(data, d => d.turnout_reg_votes))
  .domain([20,100])
  .range([margin.left, width - margin.right])

  $: xScaleTurnoutDiff = scaleLinear()
  // .domain(extent(data, d => +d.turnoutDiff))
  .domain([-25,20])
  .range([margin.left, width - margin.right])

  $: colorScale = scaleOrdinal()
  .domain(regimes)
  .range(['#99afe0', 'none', '#cf7480', 'none'])

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

  $: medianTurnout = xScaleTurnout(62);
  $: medianTurnoutDiff = xScaleTurnoutDiff(-2);

// $: console.log( data, dataCalc)

</script>

<svg width={width} height={height}>
   {#each dataCalc as d}
    <rect width='7' height='8' x={d.x} y={d.y}  fill={d.color} stroke='#191919' stroke-width={d.mandatoryVoting ? '2' : '1'}></rect>
  {/each}
  {#if xTicks === 'turnout'}
    <!-- <text  class="chart-title"x={margin.left-40} y={margin.top}>Voter turnout in </text>
    <text  class="chart-title"x={margin.left+90} y={margin.top}>democracies, </text>
    <text  class="chart-title"x={margin.left+200} y={margin.top}>authoritarian regimes</text>
    <text  class="chart-title"x={margin.left+378} y={margin.top}>and hybrid regimes.</text> -->
    {#each xTicksTurnout as tick }
    <g class="tick" transform="translate({xScaleTurnout(tick)}, {height-margin.bottom})">
      <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
  {/each}
  <text  class="country-labels" x={xScaleTurnout(99)} y={(height/2)-12}>Laos</text>
  <line  x1={xScaleTurnout(99.3)} x2={xScaleTurnout(100)} y1={(height/2-3)} y2={(height/2-10)} stroke='#525252'></line>
  <text  class="country-labels" x={xScaleTurnout(98)} y={(height/2)+30}>Vietnam</text>
  <line  x1={xScaleTurnout(98.4)} x2={xScaleTurnout(98.4)} y1={(height/2+12)} y2={(height/2+20)} stroke='#525252'></line>
  <text  class="country-labels" x={xScaleTurnout(91)} y={(height/2)-12}>Singapore</text>
  <line  x1={xScaleTurnout(96)} x2={xScaleTurnout(95)} y1={(height/2-3)} y2={(height/2-9)} stroke='#525252'></line>
  <text  class="country-labels" x={xScaleTurnout(24)} y={(height/2)-70}>Egypt had low turnout</text>
  <text  class="country-labels" x={xScaleTurnout(24)} y={(height/2)-55}>in parlamentary elections</text>
  <text  class="country-labels" x={xScaleTurnout(24)} y={(height/2)-40}>despite government's efforts</text>
  <text  class="country-labels" x={xScaleTurnout(24)} y={(height/2)-25}>to bring voters to the polls.</text>
  <line  x1={xScaleTurnout(29.3)} x2={xScaleTurnout(29.3)} y1={(height/2)-20} y2={(height/2-5)} stroke='#525252'></line>
  <text  class="country-labels" x={xScaleTurnout(17)} y={(height/2)+35}>Algeria held a referendum</text>
  <text  class="country-labels" x={xScaleTurnout(17)} y={(height/2)+50}>proposing a revised constitution.</text>
  <text  class="country-labels" x={xScaleTurnout(17)} y={(height/2)+65}>Critics called for boycott of the vote</text>
  <text  class="country-labels" x={xScaleTurnout(17)} y={(height/2)+80}>seen as a lacking in substance.</text>
  <line  x1={xScaleTurnout(23.3)} x2={xScaleTurnout(23.3)} y1={(height/2+12)} y2={(height/2+20)} stroke='#525252'></line>
  <text  class="country-labels" x={xScaleTurnout(50)} y={(height/2)+35}>Mexico formally requires</text>
  <text  class="country-labels" x={xScaleTurnout(50)} y={(height/2)+50}>voting, but imposes</text>
  <text  class="country-labels" x={xScaleTurnout(50)} y={(height/2)+65}>no formal sanctions.</text>
  <line  x1={xScaleTurnout(53.3)} x2={xScaleTurnout(53.3)} y1={(height/2+12)} y2={(height/2+20)} stroke='#525252'></line>

  <line x1='{medianTurnout+3}' x2='{medianTurnout+3}' y1='{height/2-10}' y2='{height/2+20}' stroke='#525252' stroke-linecap="round"  stroke-dasharray="4" stroke-width='2'></line>

  <g transform="translate({medianTurnout-10},50)">
    <path d="M13,80 C10,80 20,60 30,65" fill="none" stroke="#525252"></path>
  </g>
  <text  class="country-labels" x={xScaleTurnout(64.5)} y="120">median</text>
   {/if}
  {#if xTicks === 'difference'}
    <text  class="chart-title" x={margin.left-40} y={margin.top*2}>Difference in turnout in comparison to the previous election</text>
  <g>
    {#each xTicksTurnoutDiff as tick}
    <g class="tick" transform="translate({xScaleTurnoutDiff(tick)}, {height-margin.bottom})">
      <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
  {/each}
    <text  class="country-labels" x={xScaleTurnoutDiff(16.5)} y={(height/2)-12}>CAR</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(14)} y={(height/2)-12}>Burundi</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(11)} y={(height/2)+35}>In Poland's</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(11)} y={(height/2)+50}>presendential election</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(11)} y={(height/2)+65}>turnout was the</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(11)} y={(height/2)+80}>highest since 1989</text>
    <line  x1={xScaleTurnoutDiff(13.15)} x2={xScaleTurnoutDiff(13.15)} y1={(height/2+12)} y2={(height/2+20)} stroke='#525252'></line>
    <text  class="country-labels" x={xScaleTurnoutDiff(6.80)} y={(height/2)-67}>South Korea</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(6.80)} y={(height/2)-52}>saw the highest</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(6.80)} y={(height/2)-37}>turnout in a parliamentary</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(6.80)} y={(height/2)-22}>election since 1992</text>
    <line  x1={xScaleTurnoutDiff(9.15)} x2={xScaleTurnoutDiff(9.15)} y1={(height/2-5)} y2={(height/2-17)} stroke='#525252'></line>
    <text  class="country-labels" x={xScaleTurnoutDiff(-25.5)} y={(height/2)+28}>Syria</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(-25.5)} y={(height/2)-12}>Iran</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(-14.5)} y={(height/2)-37}>Dominican</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(-14.3)} y={(height/2)-22}>Republic</text>
    <line  x1={xScaleTurnoutDiff(-13.3)} x2={xScaleTurnoutDiff(-13.3)} y1={(height/2-5)} y2={(height/2-17)} stroke='#525252'></line>
  </g>

  <line x1='{medianTurnoutDiff+3}' x2='{medianTurnoutDiff+3}' y1='{height/2-15}' y2='{height/2+25}' stroke='#525252' stroke-linecap="round" stroke-dasharray="4" stroke-width='3'></line>

  {/if}
  <text  class="source" x={margin.left} y={height - (margin.bottom/2)}>Source: International IDEA Voter Turnout <a href="https://www.idea.int/data-tools/data/voter-turnout" target="_blank">Database</a></text>
  <text  class="source" x={margin.left} y={height - (margin.bottom/4.5)}>Referendums were excluded</text>
  <rect width='7' height='8' x='68%' y='10%' fill='{backgroundColor}' stroke='#191919' stroke-width='1'></rect>
  <text  class="country-labels" x='69.5%' y='13%'>One rectangle represents one election</text>
  <rect width='7' height='8' x='68%' y='15%' fill='{backgroundColor}' stroke='#191919' stroke-width='2'></rect>
  <text  class="country-labels" x='69.5%' y='18%'>Compulsory voting</text>

</svg>

<style>

.country-labels {
  font-size: 0.9rem;
}

.chart-title {
  font-size: 1.2rem;
  font-weight: bold;
  fill: rgb(66, 66, 66);
}

.source {
 color: rgb(97, 94, 92);
 font-size: 0.7rem;
}

a {
 color: rgb(97, 94, 92);
 font-size: 0.7rem;
 text-decoration: underline;
}

</style>