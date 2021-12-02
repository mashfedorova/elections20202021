<script>
  import { forceSimulation, forceX, forceY, forceCollide, scaleLinear, scaleOrdinal} from 'd3';
  import { _ } from 'lodash';
  export let data;
  export let width;
  export let x = 'turnout';
  export let xTicks;


let margin;
$: if (width < 920) {
  margin= { top:25, right:25, bottom:40, left:10 };
} else {
  margin= { top:25, right:200, bottom:40, left:200 };
}

  const continents = ["Europe","Middle East","Africa","Americas","Asia-Pacific"];
  const regimes = ["democracy","unclassified","authoritarian regime","hybrid regime"];
  const backgroundColor = "#f0e8e5";
  const xTicksTurnout = [0,10,20,30, 40, 50, 60, 70, 80, 90, 100];

  $: xTicksTurnoutDiff = [-25, -15, -5, 0, 5, 15];

  $: height = width/4;

  $: xScaleTurnout = scaleLinear()
  // .domain(extent(data, d => d.turnout_reg_votes))
  .domain([0,100])
  .range([margin.left, width - margin.right])

  $: xScaleTurnoutDiff = scaleLinear()
  // .domain(extent(data, d => +d.turnoutDiff))
  .domain([-25,20])
  .range([margin.left, width - margin.right])

  $: colorScale = scaleOrdinal()
  .domain(regimes)
  .range(["#f26b6b",backgroundColor,"#704bdd",backgroundColor])

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

  $: medianTurnout = xScaleTurnout(61);
  $: medianTurnoutDiff = xScaleTurnoutDiff(-3);

</script>

<svg width={width} height={height}>
  {#each dataCalc as d}
    <rect width='7' height='8' x={d.x} y={d.y}  fill={d.color} stroke='#191919' stroke-width={d.mandatoryVoting ? '2' : '1'}></rect>
  {/each}
  {#if width > 1100 && xTicks === 'turnout'}
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
  <text  class="country-labels" x={xScaleTurnout(24)} y={(height/2)-70}>Egypt had a low turnout</text>
  <text  class="country-labels" x={xScaleTurnout(24)} y={(height/2)-55}>in parliamentary elections</text>
  <text  class="country-labels" x={xScaleTurnout(24)} y={(height/2)-40}>despite  the government's efforts</text>
  <text  class="country-labels" x={xScaleTurnout(24)} y={(height/2)-25}>to bring voters to the polls</text>
  <line  x1={xScaleTurnout(29.3)} x2={xScaleTurnout(29.3)} y1={(height/2)-20} y2={(height/2-5)} stroke='#525252'></line>
  <text  class="country-labels" x={xScaleTurnout(17)} y={(height/2)+35}>Algeria held a referendum</text>
  <text  class="country-labels" x={xScaleTurnout(17)} y={(height/2)+50}>proposing a revised constitution.</text>
  <text  class="country-labels" x={xScaleTurnout(17)} y={(height/2)+65}>Critics called for a boycott of the vote</text>
  <text  class="country-labels" x={xScaleTurnout(17)} y={(height/2)+80}>seen as lacking in substance</text>
  <line  x1={xScaleTurnout(23.3)} x2={xScaleTurnout(23.3)} y1={(height/2+12)} y2={(height/2+20)} stroke='#525252'></line>
  <text  class="country-labels" x={xScaleTurnout(50)} y={(height/2)+40}>Mexico formally requires</text>
  <text  class="country-labels" x={xScaleTurnout(50)} y={(height/2)+55}>voting, but imposes</text>
  <text  class="country-labels" x={xScaleTurnout(50)} y={(height/2)+70}>no formal sanctions</text>
  <line  x1={xScaleTurnout(54.5)} x2={xScaleTurnout(54.5)} y1={(height/2+15)} y2={(height/2+25)} stroke='#525252'></line>
  <text  class="country-labels" x={xScaleTurnout(5)} y={(height/2)-25}>Referendum in</text>
  <text  class="country-labels" x={xScaleTurnout(5)} y={(height/2)-12}>Mexico</text>
  <line  x1={xScaleTurnout(7.5)} x2={xScaleTurnout(7.5)} y1={(height/2-3)} y2={(height/2-9)} stroke='#525252'></line>


  <line x1='{medianTurnout+3}' x2='{medianTurnout+3}' y1='{height/2-10}' y2='{height/2+20}' stroke='#525252' stroke-linecap="round" stroke-width='3'></line>

  <g transform="translate({medianTurnout-7},45)">
    <path d="M10,93 L10,90 C10,75 20,70 30,70 L 37,70" fill="none" stroke="#525252"></path>
  </g>
  <text  class="country-labels" x={xScaleTurnout(64)} y="119">median</text>

  <rect width='7' height='8' x='68%' y='10%' fill='{backgroundColor}' stroke='#191919' stroke-width='1'></rect>
  <text  class="country-labels" x='69.5%' y='13%'>One rectangle represents one election</text>
  <rect width='7' height='8' x='68%' y='15%' fill='{backgroundColor}' stroke='#191919' stroke-width='2'></rect>
  <text  class="country-labels" x='69.5%' y='18%'>Compulsory voting</text>
   {/if}


  {#if width > 1100 && xTicks === 'difference'}
  <g>
    {#each xTicksTurnoutDiff as tick}
    <g class="tick" transform="translate({xScaleTurnoutDiff(tick)}, {height-margin.bottom})">
      <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
  {/each}
    <text  class="country-labels" x={xScaleTurnoutDiff(16.8)} y={(height/2)-12}>CAR</text>
    <text  class="country-labels" x={xScaleTurnoutDiff(13.5)} y={(height/2)-12}>Burundi</text>
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

  <line x1='{medianTurnoutDiff+3}' x2='{medianTurnoutDiff+3}' y1='{height/2-15}' y2='{height/2+25}' stroke='#525252' stroke-linecap="round" stroke-width='3'></line>

  {/if}

  {#if width < 920 && width > 500 && xTicks === 'difference'}
    {#each xTicksTurnoutDiff as tick}
    <g class="tick" transform="translate({xScaleTurnoutDiff(tick)}, {height-margin.bottom})">
      <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
  {/each}
  {/if}


  {#if width < 920 && width > 500 && xTicks === 'turnout'}
    {#each xTicksTurnout as tick }
    <g class="tick" transform="translate({xScaleTurnout(tick)}, {height-margin.bottom})">
      <text x="0" y="0" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
  {/each}
  {/if}

  {#if width < 500 && xTicks === 'turnout'}
    {#each xTicksTurnout as tick }
    <g class="tick" transform="translate({xScaleTurnout(tick)}, {height-margin.bottom})">
      <text x="0" y="20" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
    {/each}
  {/if}

    {#if width < 500 && xTicks === 'difference'}
    {#each xTicksTurnoutDiff as tick }
    <g class="tick" transform="translate({xScaleTurnoutDiff(tick)}, {height-margin.bottom})">
      <text x="0" y="20" fill='#000' opacity='0.7' fade:in>{tick}</text>
    </g>
  {/each}
  {/if}

  {#if width>500}
  <text  class="source" x={margin.left} y={height - (margin.bottom/2)}>Source: International IDEA Voter Turnout <a href="https://www.idea.int/data-tools/data/voter-turnout" target="_blank">Database</a></text>
  <text  class="source" x={margin.left} y={height - (margin.bottom/4.5)}>Referendums were excluded</text>
  {/if}


</svg>

<style>

.country-labels {
  font-size: 0.9rem;
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