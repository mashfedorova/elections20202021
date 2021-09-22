<script>
  import { line, scaleLinear, scaleTime, extent, max, curveBumpY } from 'd3';
  import { fade } from 'svelte/transition';

  export let data;
  export let width;
  export let height;
  export let constData;

  const margin = { top:20, right:100, bottom:30, left: 100 };

  $: xScale = scaleTime()
    .domain(extent(constData, d => d.heldMonthYearDate))
    .range([margin.left, width-margin.right ])

  $: yScale = scaleLinear()
    .domain([0, max(constData, d => d.position)])
    .range([height-margin.bottom, margin.top])

  const stripes = line()
    .x(d => xScale(d.heldMonthYearDate)+10)
    .y(d => yScale(d.position)+20)
    .curve(curveBumpY)

</script>
  <path
  transition:fade
  d={stripes(data)}
  stroke='#8f8885'
  fill= 'none'
  stroke-width='2'
  opacity='0.4'
  ></path>

<style>
</style>