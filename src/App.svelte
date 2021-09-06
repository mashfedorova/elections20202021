<!-- <svelte:head>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.6.1/gsap.min.js"></script>
  <script src="https://scdnjs.cloudflare.com/ajax/libs/gsap/3.6.1/ScrollTrigger.min.js"></script>
  <script src=".scripts/DrawSVGPlugin.min.js"></script>
</svelte:head> -->
<script>
import { gsap } from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { csv, timeParse} from 'd3';
import { onMount } from 'svelte';
import IntroChart from '../components/IntroChart.svelte'
gsap.registerPlugin(ScrollTrigger);
let width = document.body.clientWidth;
$: widthChart = width > 1270 ? 1270 : width;
let updatedData = [];
let data = [];
let cancelledBallots = [];
const parseMonthYear = timeParse("%B %Y");
let whichY = 'position';
function resize() {
   width = document.body.clientWidth
}
function updateData() {
  updatedData = data.filter(d => d.heldAtAll === 'yes');
}
function step2() {
  cancelledBallots = data.filter(d => d.heldAtAll === 'no');
}

function step3() {
  whichY = 'test';
}
onMount(
  async () => {
  const dataRaw = await csv('data/dataFinal.csv', d => {
    return {
    ...d,
    position: +d.position,
    test: +d.position + 1,
    heldMonthYearDate: parseMonthYear(d.heldMonthYear),
    postponedMonthYearDate: parseMonthYear(d.postponedMonthYear),
    }
  })
  data = dataRaw;
  console.log(data)
});
console.log(whichY)
onMount(resize)
onMount(() => {
  ScrollTrigger.create({
    trigger: '#intro-chart',
    endTrigger: '#step-3',
    start: 'center center',
    end: '#step-3',
    pin: true,
    pinSpacing: false,
  });
  ScrollTrigger.create({
      trigger: "#step-1",
      start: "top 80%",
      });
  ScrollTrigger.create({
      trigger: "#intro-chart",
      start: "center 80%",
      markers: true,
      onEnter: updateData
});
  ScrollTrigger.create({
      trigger: "#step-2",
      start: "top 80%",
      onEnter: step2
      });

  ScrollTrigger.create({
      trigger: "#step-3",
      start: "top 80%",
      onEnter: step3
      });
})
</script>
<svelte:window on:resize='{resize}'/>
<section class="intro">
  <h1> Elections between 2020 and July 2021</h1>
  <p class="intro-para">Lorem ipsum dolor sit amet consectetur adipisicing elit. Molestiae ea tenetur exercitationem, voluptatem quo temporibus aliquid optio suscipit, veniam pariatur quaerat natus at. Ab vitae iusto dolor voluptas officiis eaque libero reiciendis quas voluptates perferendis ratione, odio corrupti omnis tempora quo a? Amet ducimus laborum ipsum quidem consequatur, vitae ex nihil dolore accusamus tempora, iusto hic culpa vero, doloremque eos.</p>
</section>
<div id="intro-chart" bind:this={width} >
  <IntroChart data={updatedData} width={widthChart} {cancelledBallots} {whichY}/>
  <!-- <CancelledBallots {cancelledBallots} width={widthChart}/> -->

</div>
<article class="scrolls">
  <section class="step" id="step-1">Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet, fugiat dolores. Quo officia facere tenetur dicta eligendi dolore esse consectetur eos quis iusto eveniet nesciunt, dolorum aliquam repudiandae dignissimos provident!</section>
  <section class="step" id="step-2">Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet, fugiat dolores. Quo officia facere tenetur dicta eligendi dolore esse consectetur eos quis iusto eveniet nesciunt, dolorum aliquam repudiandae dignissimos provident!</section>
  <section class="step" id="step-3">Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet, fugiat dolores. Quo officia facere tenetur dicta eligendi dolore esse consectetur eos quis iusto eveniet nesciunt, dolorum aliquam repudiandae dignissimos provident!</section>
</article>

<style>
.pin-spacer {
  align-items: center;
  justify-content: center;
}
.intro {
  margin: 0 auto;
  max-width: 650px;
}
.intro-para {
text-align: center;
font-size: 1.2rem;
}
h1 {
  text-align: center;
  margin-top: 40rem;
  margin-bottom: 2rem;
  font-size: 2.5rem;
}
#intro-chart {
  margin: 0 auto;
  /* max-width: 1200px; */
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
 .step {
  border: 2px solid #000;
  box-shadow: 3px 3px 0 0 rgb(165, 165, 165);
  margin: 0 auto;
  width: 55%;
  max-width: 480px;
  margin-bottom: 85vh;
  margin-top: 20vh;
  position: relative;
  background-color: #fff;
  padding: 1em;
  line-height: 1.4;
  color: #414141;
  font-size: 1.2rem;
}
#step-1 {
  margin-top: 0;
}
/*
 scroll-text__div {
    margin-left: 0;
    background-color: #fff;
    border: 2px solid #000;
    box-shadow: 4px 4px 0 0 #767676;
    padding: 6px 10px;
    padding-bottom: 0;
    font-size: 28px;
    line-height: 32px;
    font-weight: 400;
    font-size: 20px;
    line-height: 26px;
    padding-top: 4px;
} */
</style>