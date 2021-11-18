<script>
  import { select, scaleOrdinal, map } from 'd3';
  import { afterUpdate } from 'svelte/internal';
  import  {_} from 'lodash';

  export let width;
  export let data;

  const backgroundColor = "#f0e8e5";

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

  let height
  $: if (width < 600) {
    height = width;
  } else {
    height = width/2;
  }

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

   afterUpdate(() => {
    let svg = select('#incumbentsSvg')
    let ballotsDem  = svg.selectAll('.democ')
      .data(democracies)
    let ballotsAut  = svg.selectAll('.autocr')
      .data(autocracies)
    let ballotsHyb = svg.selectAll('.hybrids')
      .data(hybrids)

         ballotsDem.on("mouseenter", (event, d) => {
        tooltip
          .style("visibility", "visible")
          .style("left", event.pageX + "px")
          .style("top", event.pageY + "px")
          .html(
            `<div>${d.country}</div>`
          )
          })

          ballotsDem.on("mouseout", (event, d) => {
        tooltip
          .style("visibility", "hidden")
          })

         ballotsAut.on("mouseenter", (event, d) => {
        tooltip
          .style("visibility", "visible")
          .style("left", event.pageX + "px")
          .style("top", event.pageY + "px")
          .html(
            `<div>${d.country}</div>`
          )
          })

          ballotsAut.on("mouseout", (event, d) => {
        tooltip
          .style("visibility", "hidden")
          })

         ballotsHyb.on("mouseenter", (event, d) => {
        tooltip
          .style("visibility", "visible")
          .style("left", event.pageX + "px")
          .style("top", event.pageY + "px")
          .html(
            `<div>${d.country}</div>`
          )
          })

          ballotsHyb.on("mouseout", (event, d) => {
        tooltip
          .style("visibility", "hidden")
          })


  })


</script>

<svg {width} {height} id="incumbentsSvg">
  <!-- <text class="regimes" x="100" y="70">democracies</text>
  <text class="regimes" x="450" y="70">athoritarian regimes</text>
  <text class="regimes" x="800" y="70">hybrid regimes</text> -->

  {#if width > 1119}
    <g transform="translate(100,100)">
      <text class="regimes" x="0" y="-10">democracies</text>
    </g>
    <g transform="translate(450,100)">
      <text class="regimes" x="0" y="-10">hybrid regimes</text>
    </g>
    <g transform="translate(800,100)">
      <text class="regimes" x="0" y="-10">athoritarian regimes</text>
    </g>
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
        class="democ"
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
        class="hybrids"
        x="3"
        y="5"
        width="18"
        height="23"
        fill="{colorScale(d.incumbentWon)}"
        opacity="0.8"
        filter='url(#highlight)'
        >
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
        class="autocr"
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
  {/if}
  {#if width < 1120 && width > 920}
  <g transform="translate(300,50)">
    <text class="regimes" x="0" y="-10">democracies</text>
  </g>
    <g transform="translate(300,190)">
      <text class="regimes" x="0" y="-10">hybrid regimes</text>
    </g>
    <g transform="translate(300,300)">
      <text class="regimes" x="0" y="-10">athoritarian regimes</text>
    </g>
   {#each democracies as d, i}
    <g transform="translate(300,50)">
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
    <g transform="translate(300,190)">
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
    <g transform="translate(300,300)">
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
        filter='url(#highlight)'
        >
        </rect>
      </g>
</g>
  {/each}


  {/if}

   {#if width < 920}
  <g transform="translate(50,50)">
    <text class="regimes" x="0" y="-10">democracies</text>
  </g>
  <g transform="translate(50,190)">
      <text class="regimes" x="0" y="-10">hybrid regimes</text>
    </g>
    <g transform="translate(50,300)">
      <text class="regimes" x="0" y="-10">athoritarian regimes</text>
    </g>
   {#each democracies as d, i}
    <g transform="translate(50,50)">
      <g transform="translate({(i % 10) * 28},{Math.floor(i / 10) * 20})">
        <rect
        x="3"
        y="5"
        width="7"
        height="8"
        fill="{colorScale(d.incumbentWon)}"
        opacity="0.8"
        stroke='#000'
        >
        </rect>
      </g>
</g>
  {/each}

    {#each hybrids as d, i}
    <g transform="translate(50,190)">
      <g transform="translate({(i % 10) * 28},{Math.floor(i / 10) * 20})">
        <rect
        x="3"
        y="5"
        width="7"
        height="8"
        fill="{colorScale(d.incumbentWon)}"
        opacity="0.8"
        stroke='#000'
        >
        </rect>
      </g>
</g>
  {/each}

    {#each autocracies as d, i}
    <g transform="translate(50,300)">
      <g transform="translate({(i % 10) * 28},{Math.floor(i / 10) * 20})">
        <rect
        x="3"
        y="5"
        width="7"
        height="8"
        fill="{colorScale(d.incumbentWon)}"
        opacity="0.8"
        stroke='#000'
        >

        </rect>
      </g>
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
  .regimes {
    letter-spacing: 0.2em;
  }
</style>