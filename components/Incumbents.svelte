<script>
  import { scaleLinear, extent, scaleOrdinal, map } from 'd3';
  import  {_} from 'lodash';

  export let width;
  export let data;

  const backgroundColor = "#f0e8e5";

  $: height = width/2;
  $: democracies = data.filter(d => d.demIndexCat === 'democracy');
  $: autocracies = data.filter(d => d.demIndexCat === 'authoritarian regime');
  $: hybrids = data.filter(d => d.demIndexCat === 'hybrid regime');

  // const margin= { top:25, right:200, bottom:60, left:200 };

  // $: dataCalc = _.chain(data)
  //   .sortBy('incumbentWon')
  //   .value();

  $: colorScale = scaleOrdinal()
  .domain(map(data, d => d.incumbentWon))
  .range(["#709afa", "#dbbd84", "#949494"]);


</script>

<svg {width} {height}>
  <text class="regimes" x="100" y="70">democracies</text>
  <text class="regimes" x="450" y="70">athoritarian regimes</text>
  <text class="regimes" x="800" y="70">hybrid regimes</text>
  {#each democracies as d, i}
    <g transform="translate(100,100)">
      <g transform="translate({(i % 10) * 28},{Math.floor(i / 10) * 35}) scale(0.8)">
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
        >
        </circle>
        <rect
        x="3"
        y="5"
        width="18"
        height="23"
        fill="{colorScale(d.incumbentWon)}"
        opacity="0.8"
        filter='url(#highlight)'>
        </rect>
      </g>
</g>
  {/each}

    {#each hybrids as d, i}
    <g transform="translate(450,100)">
      <g transform="translate({(i % 10) * 28},{Math.floor(i / 10) * 35}) scale(0.8)">
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
        <rect
        x="3"
        y="5"
        width="18"
        height="23"
        fill="{colorScale(d.incumbentWon)}"
        opacity="0.8"
        filter='url(#highlight)'>
        </rect>
      </g>
</g>
  {/each}

    {#each autocracies as d, i}
    <g transform="translate(800,100)">
      <g transform="translate({(i % 10) * 28},{Math.floor(i / 10) * 35}) scale(0.8)">
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
        <rect
        x="3"
        y="5"
        width="18"
        height="23"
        fill="{colorScale(d.incumbentWon)}"
        opacity="0.8"
        filter='url(#highlight)'>
        </rect>
      </g>
</g>
  {/each}

  <defs>
    <filter id="highlight">
      <feGaussianBlur stdDeviation="7 7"/>
    </filter>
  </defs>
</svg>

<style>
  .regimes {
    letter-spacing: 0.2em;
  }
</style>