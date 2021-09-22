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
import IntroChart from '../components/IntroChart.svelte';
import  {_} from 'lodash';
gsap.registerPlugin(ScrollTrigger);

let width = document.body.clientWidth;
$: widthChart = width > 1270 ? 1270 : width;

let updatedData = [];
let data = [];
let cancelledBallots = [];
let dataRectsPostponed = [];
let whichY = 'y';
let whichX = 'x';
const parseMonthYear = timeParse("%B %Y");
let monthsRaw = [];
let regimes = [];
let monthsList = ["April 2021","June 2021","November 2020","June 2020","August 2020","October 2020","December 2020","March 2021","July 2020",
"May 2021","September 2020","February 2021","March 2020","January 2021","April 2020","May 2020"];
let dataPostponedBallots = [];
let marginLeft = 100;
let colorBallot = 'no';
let highlighApr = 'no';

function resize() {
   width = document.body.clientWidth
}

function updateData() {
  updatedData = data.filter(d => d.heldAtAll === 'yes');
  monthsRaw = monthsList;
}

function step2() {
  highlighApr = 'yes';
}

function step3() {
  highlighApr === 'cancelled';
  cancelledBallots = data.filter(d => d.heldAtAll === 'no');
}

function step4() {
  highlighApr = 'no';
  whichY =  'yDem';
  whichX = 'xDem';
  data = _.sortBy(data, ['demIndexCat', 'heldDiff'])
  dataRectsPostponed = data.filter(d => d.werePostponed === 'yes' && d.heldAtAll === 'yes');
  monthsRaw = [];
  regimes = ["democracy","unclassified","authoritarian regime","hybrid regime"];
}

function step5() {
  whichY =  'yHeld';
  whichX = 'xHeld';
  updatedData = dataRectsPostponed
  monthsRaw = monthsList
  cancelledBallots = []
  dataPostponedBallots = data.filter(d => d.werePostponed === 'yes' && d.heldAtAll === 'yes');
  regimes = [];
  marginLeft = 300;
  colorBallot = 'yes';

}

onMount(
  async () => {
  const dataRaw = await csv('data/dataFinal7.csv', d => {
    return {
    ...d,
    position: +d.position,
    demIndexScaleY: +d.demIndexScaleY,
    demIndexScaleX: +d.demIndexScaleX,
    heldMonthYearDate: parseMonthYear(d.heldMonthYear),
    postponedMonthYearDate: parseMonthYear(d.postponedMonthYear),
    postponedMonthYearDate2: parseMonthYear(d.postponedMonthYear2),
    heldDiff: d.heldMonthYearDate - d.postponedMonthYearDate,
    heldDiff: +d.heldDiff
    }
  })
  data = dataRaw;
});

onMount(resize)

onMount(() => {

  ScrollTrigger.create({
    trigger: '#intro-chart',
    endTrigger: '#step-6',
    start: 'center center',
    end: '#step-6',
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

  ScrollTrigger.create({
      trigger: "#step-4",
      start: "top 80%",
      onEnter: step4
      });

  ScrollTrigger.create({
      trigger: "#step-5",
      start: "top 80%",
      onEnter: step5
      });
})
</script>
<svelte:window on:resize='{resize}'/>
<section class="intro">
  <h1> Elections between 2020 and July 2021</h1>
  <!-- <h1> Text</h1> -->
  <p class="intro-para">Between March 2020, when WHO has <a href='https://www.euro.who.int/en/health-topics/health-emergencies/coronavirus-covid-19/news/news/2020/3/who-announces-covid-19-outbreak-a-pandemic'>declared</a> coronavirus (COVID-19) outbreak a global pandemic and June 2021. Between March 2020 and June 2021, elections were held in 75 countries. In total, 88 elections were held, with 12 countries having held multiple elections during this period. Lorem ipsum dolor sit amet consectetur adipisicing elit. Molestiae ea tenetur exercitationem, voluptatem quo temporibus aliquid optio suscipit, veniam pariatur quaerat natus at. Ab vitae iusto dolor voluptas officiis eaque libero reiciendis quas voluptates perferendis ratione, odio corrupti omnis tempora quo a? Amet ducimus laborum ipsum quidem consequatur, vitae ex nihil dolore accusamus tempora, iusto hic culpa vero, doloremque eos.</p>
</section>
<div id="intro-chart" bind:this={width} >
  <IntroChart data={updatedData} width={widthChart} {cancelledBallots} {whichY} {whichX} {dataRectsPostponed} {monthsRaw} {regimes}
  constData={data} {dataPostponedBallots} {marginLeft} {colorBallot} {highlighApr}/>
  <!-- <CancelledBallots {cancelledBallots} width={widthChart}/> -->

</div>
<article class="scrolls">
  <section class="step" id="step-1">
    <p>Between March 2020 and June 2021, 88 elections were held in 75 countries, with 12 countries having held multiple elections during this period. Sixteen countries at some point had <span class="postponed">postponed</span> the election.</p>
    <p>One ballot<svg width="28" transform="scale(0.6)" class="ballot-inline"><path d="M23.3,31C16.2,31,9,30.8,1.8,30.2C1.2,20.7,1.3,11.2,2,1.7c7.1-0.1,14.2,0.3,21.2,1C24.1,12.1,24.1,21.6,23.3,31z M12.7,12.3c2.8-0.4,5.6-0.6,8.5-0.5 M13.5,21.3l7.2,0.2 M5.9,10.7l1.9,3.4l3.6-7.6" stroke="#000" fill="#f0e8e5" stroke-width="2.7"></path><circle cx="8" cy="23" r="3" stroke="#000" fill="#f0e8e5" stroke-width="2"></circle></svg>represents one national election (either presedential or parliamentary), held on a given day and two ballots<svg width="29" transform="scale(0.6)" class="ballot-inline ballot-two"><path d="M23.3,28.1c-5.5,0.7-11.1,0.1-16.3-1.6c1-8.7,2.3-17,4-25c5.7-0.3,11.4,0.2,17,2.5C27.4,11.9,26.3,19.7,23.3,28.1z" stroke="#000" fill="#f0e8e5" stroke-width="2.5"></path><path d="M22.2,30.6c-6.3,1-12.7,1.8-19.1,2.3C1.6,24,1,15.1,1.4,6.2C7.3,5.6,13.1,5.1,19,4.9C19.8,13.4,20.5,21.9,22.2,30.6z M4.4,11.7c0.6,0.6,1.2,1.2,1.8,1.8c0.4-1.5,1.1-2.8,2-4 M11.6,11.4c1.1-0.4,2.3-0.6,3.6-0.6 M13.3,22.6c1.2-0.4,2.4-0.5,3.6-0.4" stroke="#000" fill="#f0e8e5" stroke-width="2.5"></path><circle cx="8" cy="23" r="3" stroke="#000" fill="#f0e8e5" stroke-width="2.5"></circle></svg>represent countries where citizens elected president and parliament on the same day. </p>
  </section>
  <section class="step" id="step-2">Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet, fugiat dolores. Quo officia facere tenetur dicta eligendi dolore esse consectetur eos quis iusto eveniet nesciunt, dolorum aliquam repudiandae dignissimos provident!</section>
  <section class="step" id="step-3">Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet, fugiat dolores. Quo officia facere tenetur dicta eligendi dolore esse consectetur eos quis iusto eveniet nesciunt, dolorum aliquam repudiandae dignissimos provident!</section>
  <section class="step" id="step-4">Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet, fugiat dolores. Quo officia facere tenetur dicta eligendi dolore esse consectetur eos quis iusto eveniet nesciunt, dolorum aliquam repudiandae dignissimos provident!</section>
  <section class="step" id="step-5">Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet, fugiat dolores. Quo officia facere tenetur dicta eligendi dolore esse consectetur eos quis iusto eveniet nesciunt, dolorum aliquam repudiandae dignissimos provident!</section>
  <section class="step" id="step-6">Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet, fugiat dolores. Quo officia facere tenetur dicta eligendi dolore esse consectetur eos quis iusto eveniet nesciunt, dolorum aliquam repudiandae dignissimos provident!</section>
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
  max-width: 550px;
  margin-bottom: 85vh;
  margin-top: 20vh;
  position: relative;
  background-color: #f0e8e5;
  padding: 1em;
  line-height: 1.4;
  color: #414141;
  font-size: 1.2rem;
}
#step-1 {
  margin-top: 0;
}

.postponed {
  background: rgb(112, 153, 250, 0.4);
  background-repeat: no-repeat;
  background-size: 100% 90%;
  background-position: 0 100%;
}

.ballot-inline {
  max-height: 1em;
  display: inline;
  position: relative;
  vertical-align: top;
  overflow: visible;
  /* bottom: 0.50em; */
  /* top: 0.60em; */

}

.ballot-two {
   /* bottom: 0.90em; */
   /* top: 0.60em; */
   /* vertical-align: bottom; */


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