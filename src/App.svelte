<script>
import { csv } from 'd3';
import { onMount } from 'svelte';

let data = [];

onMount(async () => {
  const dataRaw = await csv('data/dataFinal.csv', d => {
    return {
    ...d,
    position: +d.position
    }
  })
  data = dataRaw;
  console.log(data)
});
</script>


<style>

</style>