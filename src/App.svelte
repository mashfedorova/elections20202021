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
import TurnoutChart from '../components/TurnoutChart.svelte';
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
let legend = 'no';
let opacityValue = 'opacity';
let highlightLines = 'no';
let xTurnout = 'turnout';
let xTurnoutDiff = 'turnoutDiff';
let dataTurnout = [];
let dataTurnoutDiff = [];
let xTicksTurn = 'turnout';
let xTicksDiff = 'difference';

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
  opacityValue = 'opFadeout';
  cancelledBallots = data.filter(d => d.heldAtAll === 'no');
}

function step4() {
  opacityValue = 'opacity';
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
  updatedData = dataRectsPostponed;
  monthsRaw = monthsList;
  cancelledBallots = [];
  dataPostponedBallots = data.filter(d => d.werePostponed === 'yes' && d.heldAtAll === 'yes');
  regimes = [];
  marginLeft = 300;
  colorBallot = 'yes';
  legend = 'yes';
}

function step6() {
  highlightLines = 'yes';
  opacityValue = 'opacityLines';
}

onMount(
  async () => {
  const dataRaw = await csv('data/dataFinal9.csv', d => {
    return {
    ...d,
    position: +d.position,
    demIndexScaleY: +d.demIndexScaleY,
    demIndexScaleX: +d.demIndexScaleX,
    heldMonthYearDate: parseMonthYear(d.heldMonthYear),
    postponedMonthYearDate: parseMonthYear(d.postponedMonthYear),
    postponedMonthYearDate2: parseMonthYear(d.postponedMonthYear2),
    heldDiff: d.heldMonthYearDate - d.postponedMonthYearDate,
    heldDiff: +d.heldDiff,
    turnout_reg_votes: +d.turnout_reg_votes,
    turnoutDiff: d.turnoutDiff
    }
  })
  data = dataRaw;
  dataTurnout = dataRaw.filter(d => d.turnout_reg_votes)
  dataTurnoutDiff = _.chain(data.filter(d => !!d.turnoutDiff))
    .map((d) => {
      return {
        turnoutDiff: d.turnoutDiff,
        turnout_reg_votes: d.turnout_reg_votes,
        country: d.country,
        continent: d.continent,
        demIndexCat: d.demIndexCat

      }
    }).value()
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

  ScrollTrigger.create({
      trigger: "#step-6",
      start: "top 80%",
      onEnter: step6,
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
  constData={data} {dataPostponedBallots} {marginLeft} {colorBallot} {highlighApr} {legend} {opacityValue} {highlightLines}/>
  <!-- <CancelledBallots {cancelledBallots} width={widthChart}/> -->
</div>
<article class="scrolls">
  <section class="step" id="step-1">
    <p>Between March 2020 and June 2021, 89 elections were held in 76 countries, with 12 countries having held multiple elections during this period. Sixteen countries at some point had <span class="postponed">postponed</span> the election.</p>
    <p>One ballot<svg width="28" transform="scale(0.6)" class="ballot-inline"><path d="M23.3,31C16.2,31,9,30.8,1.8,30.2C1.2,20.7,1.3,11.2,2,1.7c7.1-0.1,14.2,0.3,21.2,1C24.1,12.1,24.1,21.6,23.3,31z M12.7,12.3c2.8-0.4,5.6-0.6,8.5-0.5 M13.5,21.3l7.2,0.2 M5.9,10.7l1.9,3.4l3.6-7.6" stroke="#000" fill="#f0e8e5" stroke-width="2.7"></path><circle cx="8" cy="23" r="3" stroke="#000" fill="#f0e8e5" stroke-width="2"></circle></svg>represents one national election (either presedential or parliamentary), held on a given day and two ballots<svg width="29" transform="scale(0.6)" class="ballot-inline ballot-two"><path d="M23.3,28.1c-5.5,0.7-11.1,0.1-16.3-1.6c1-8.7,2.3-17,4-25c5.7-0.3,11.4,0.2,17,2.5C27.4,11.9,26.3,19.7,23.3,28.1z" stroke="#000" fill="#f0e8e5" stroke-width="2.5"></path><path d="M22.2,30.6c-6.3,1-12.7,1.8-19.1,2.3C1.6,24,1,15.1,1.4,6.2C7.3,5.6,13.1,5.1,19,4.9C19.8,13.4,20.5,21.9,22.2,30.6z M4.4,11.7c0.6,0.6,1.2,1.2,1.8,1.8c0.4-1.5,1.1-2.8,2-4 M11.6,11.4c1.1-0.4,2.3-0.6,3.6-0.6 M13.3,22.6c1.2-0.4,2.4-0.5,3.6-0.4" stroke="#000" fill="#f0e8e5" stroke-width="2.5"></path><circle cx="8" cy="23" r="3" stroke="#000" fill="#f0e8e5" stroke-width="2.5"></circle></svg>represent countries where citizens elected president and parliament on the same day. </p>
  </section>
  <section class="step" id="step-2">
    <p>In April 2020, a month after WHO declared COVID-19 outbreak a pandemic many countries decided to cancel scheduled elections. Only two countries - Kiribati and South Korea - held elections nevertheless. </p>
    <p><strong>South Korea</strong> was widely <a href="https://aceproject.org/ero-en/misc/elections-and-covid-19-technical-paper-by">praised </a>for measures implemented during the elections. Early voting was encouraged, home voting provisions were extended, and stringent measures were implemented at the polling stations. Public approval of the governemnt's handling of the pandemic was evident in a landslide victory of President Moon Jae-in’s governing party in parliamentary elections.</p>
    <p><strong>Kiribati</strong> went ahead with parliamentary elections, after having postpone them for a week because of disrupted public services. At the time of elections, however, there were no official cases of COVID-19 <a href='https://www.rnz.co.nz/international/pacific-news/414192/kiribati-goes-to-polls-as-normal-despite-covid-19-threat'>recoded</a>.</p>
  </section>
  <section class="step" id="step-3">
    <p>Four countries - Chad, Haiti, Somalia and Armenia - postponed elections and to the momement of publication have not yet held these elections.</p>
    <p> Some of these countries had more reasons to postpone elections than others. Armenia declared a state of emergency, having the highest number of COVID-19 cases among countries in the South Caucasus region. Parlamentary elections in Chad, originally scheduled in 2015 had been <a href="https://www.dw.com/en/chad-tensions-rise-over-debys-presidential-run/a-56536855">repeatedly</a> postponed even before the pandemic. President Idriss Deby has ruled Chad for the past 30 years. </p>
  </section>
  <section class="step" id="step-4">
    <p>Elections were postponed in democracratic regimes, as well as in authoritarian. However, twice as many democracies postponed elections as autocracies. While democracies always held postponed elections later, non-democratic regimes had a tendency of postponing elections indefinitely. </p>
  </section>
  <section class="step" id="step-5">
    <p>Non-democratic regimes tended to postpone elections for a longer. While on average <span class="dem">democracies</span> postponed elections for approximately two months, <span class="aut">autocracies</span> and <span class="hyb">hybrid regimes</span> postponed electons on average for four months.</p>
  </section>
  <section class="step" id="step-6">
    <p><strong>Ethiopia</strong> postponed parlamentary elections mutltiple times and for the longest period, for about ten months. In the end of August, when elections were originally scheduled 1,500 COVID-19 cases were recoded in the country, one of the highest numbers during the pandemic up to this date. The <a href="https://www.aljazeera.com/opinions/2021/6/18/why-ethiopias-elections-should-be-postponed">postponement</a> of elections, done unilaterally by the countries prime minister Abiy Ahmed was met with critisism by his opponents. </p>
  </section>
  <section id="turnout-intro">Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet, fugiat dolores. Quo officia facere tenetur dicta eligendi dolore esse consectetur eos quis iusto eveniet nesciunt, dolorum aliquam repudiandae dignissimos provident!</section>
  <div id="turnout-chart" bind:this={width} >
    <TurnoutChart data={dataTurnout} width={widthChart} x={xTurnout} xTicks = {xTicksTurn}/>
  </div>
  <div id="turnout-chart" bind:this={width} >
    <TurnoutChart data={dataTurnoutDiff} width={widthChart} x={xTurnoutDiff} xTicks = {xTicksDiff}/>
  </div>
</article>

<style>
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

#turnout-chart{
  margin: 0 auto;
  /* max-width: 1200px; */
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  /* height: 50vh; */
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
  padding-left: 2px;
  padding-right: 2px;
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

.aut {
  background: rgba(217, 128, 140, 0.4) ;
  background-repeat: no-repeat;
  background-size: 100% 90%;
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}

.dem {
  background: rgba(112, 153, 250, 0.4);
  background-repeat: no-repeat;
  background-size: 100% 90%;
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}

.hyb {
  background: rgba(34, 171, 130, 0.4);
  background-repeat: no-repeat;
  background-size: 100% 90%;
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}

a {
 color: rgb(97, 94, 92)}

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