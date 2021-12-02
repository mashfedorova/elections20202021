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
import TurnoutStringency from '../components/TurnoutStringency.svelte';
import Incumbents from '../components/Incumbents.svelte';
import  {_} from 'lodash';
gsap.registerPlugin(ScrollTrigger);

let width = document.body.clientWidth;
$: widthChart = width > 1270 ? 1270 : width;
$: widthScatter = width > 1270 ? 800 : width/1.5;

let data = [];
let cancelledBallots = [];
let dataRectsPostponed = [];
let whichY = 'yPostOrigin';
let whichX = 'xPost1';
const parseMonthYear = timeParse("%B %Y");
let regimes = 'no';
let monthsList = ["April 2021","June 2021","November 2020","June 2020","August 2020","October 2020","December 2020","March 2021","July 2020",
"May 2021","September 2020","February 2021","March 2020","January 2021","April 2020","May 2020", "July 2021", "August 2021", "September 2021", "October 2021", "November 2021"];
let monthsRaw = monthsList;
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
let dataTurnoutStringency = [];
let stringentChartY = 'turnout_reg_votes';
let yTick = 'turnoutScaled';
let yTickLabel = 'turnout_reg_votes';
let regY = 'turnout_reg_votes';
let yHorisontal = 50;
let turnoutLabel = 'turnout';
let rectTurnoutStringency = '';
let regionsHighlight = '';
let dataIncumbents = [];
let dataTurnoutStringencyUpdated = [];
let updatedData = [];


function resize() {
   width = document.body.clientWidth
}

function step2() {
  // highlighApr = 'yes';
  whichX = 'x';
  whichY = 'y';
}

function step2Back() {
  whichY = 'yPostOrigin';
  whichX = 'xPost1';
}

function step3() {
  updatedData = data.filter(d => d.heldAtAll === 'yes');
}

function step3Back() {
  updatedData = data.filter(d => d.werePostponed === 'yes' && d.heldAtAll === 'yes');
}

function step4() {
  highlighApr = 'yes';
}

function step4Back() {
  highlighApr = 'no';
}

function step5() {
  opacityValue = 'opFadeout';
  cancelledBallots = data.filter(d => d.heldAtAll === 'no');
}

function step5Back() {
  opacityValue = 'opacity';
  highlighApr = 'yes';
  cancelledBallots = [];
}

function step6() {
  opacityValue = 'opacity';
  highlighApr = 'no';
  whichY =  'yDem';
  whichX = 'xDem';
  data = _.sortBy(data, ['demIndexCat', 'heldDiff'])
  dataRectsPostponed = data.filter(d => d.werePostponed === 'yes' && d.heldAtAll === 'yes');
  monthsRaw = [];
  regimes = 'yes';
}

function step6Back() {
  opacityValue = 'opFadeout';
  whichY = 'y';
  whichX = 'x';
  regimes = 'no';
  monthsRaw = monthsList;
}

function step7() {
  whichY =  'yHeld';
  whichX = 'xHeld';
  updatedData = dataRectsPostponed;
  monthsRaw = ["November 2020","July 2020","March 2021","May 2021","September 2020","January 2021","March 2020","May 2020", "July 2021", "September 2021", "November 2021"];
  cancelledBallots = [];
  dataPostponedBallots = data.filter(d => d.werePostponed === 'yes' && d.heldAtAll === 'yes');
  regimes = ["authoritarian regime","hybrid regime","democracy","unclassified"];
  marginLeft = 300;
  colorBallot = 'yes';
  legend = 'yes';
}

function step7Back() {
  whichY =  'yDem';
  whichX = 'xDem';
  updatedData = data.filter(d => d.heldAtAll === 'yes');
  data = _.sortBy(data, ['demIndexCat', 'heldDiff']);
  dataRectsPostponed = data.filter(d => d.werePostponed === 'yes' && d.heldAtAll === 'yes');
  monthsRaw = [];
  regimes = 'yes';
  colorBallot = 'no';
  legend = 'no';
  dataPostponedBallots = [];
  cancelledBallots = data.filter(d => d.heldAtAll === 'no');
  marginLeft = 100;
}

function step8() {
  highlightLines = 'yes';
  opacityValue = 'opacityLines';
}

function step8Back() {
  highlightLines = 'no';
  opacityValue = 'opacity';
}

function turnoutStep2() {
  stringentChartY = 'turnoutDiff';
  dataTurnoutStringency = dataTurnoutStringency.filter(d => !!d.turnoutDiff)
  yTickLabel = 'turnoutDiff'
  yHorisontal = 0
  turnoutLabel = 'turnout difference'
  rectTurnoutStringency = 'yes';
}

function turnoutStep2Back() {
  stringentChartY = 'turnout_reg_votes';
  dataTurnoutStringency = dataTurnoutStringencyUpdated;
  yTickLabel = 'turnout_reg_votes';
  yHorisontal = 60;
  turnoutLabel = 'turnout';
  rectTurnoutStringency = '';

}

function turnoutStep3() {
  regionsHighlight = 'yes';
  rectTurnoutStringency = '';
}

function turnoutStep3Back() {
  regionsHighlight = '';
  rectTurnoutStringency = 'yes';
}

onMount(
  async () => {
  const dataRaw = await csv('data/dataFinal12.csv', d => {
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
    turnoutDiff: d.turnoutDiff,
    positionOriginalDate: +d.positionOriginalDate
    }
  })
  data = dataRaw;
  updatedData = data.filter(d => d.werePostponed === 'yes' && d.heldAtAll === 'yes')
  dataTurnout = dataRaw.filter(d => d.turnout_reg_votes);
  dataTurnoutDiff = _.chain(data.filter(d => !!d.turnoutDiff))
    .map((d) => {
      return {
        turnoutDiff: d.turnoutDiff,
        turnout_reg_votes: d.turnout_reg_votes,
        country: d.country,
        continent: d.continent,
        demIndexCat: d.demIndexCat

      }
    }).value();

  const dataTurnoutStringencyCalc = _.chain(dataRaw.filter(d => d.turnout_reg_votes && d.stringency_index))
    .map((d) => {
      return {
        stringency_index: +d.stringency_index,
        turnout_reg_votes: d.turnout_reg_votes,
        turnoutDiff: d.turnoutDiff,
        country: d.country,
        d: d.d,
        d1: d.d1,
        d4: d.d4,
        demIndexCat: d.demIndexCat,
        continent: d.continent,
      }
    })
    .sortBy('turnoutDiff')
    .reverse()
    .value();

  dataTurnoutStringency = dataTurnoutStringencyCalc;
  dataTurnoutStringencyUpdated = dataTurnoutStringencyCalc;

    dataIncumbents = _.chain(data.filter(d => d.type !== "referendum" && d.heldAtAll === "yes"))
    .sortBy('incumbentWon')
    .map((d) => {
      return {
        incumbentWon: d.incumbentWon,
        demIndexCat: d.demIndexCat,
        country: d.country,
        d: d.d,
        d1: d.d1,
        d4: d.d4,
      }
    })
    .reverse()
    .value();

});

onMount(resize)
onMount(resize)

onMount(() => {

  ScrollTrigger.create({
    trigger: '#intro-chart',
    endTrigger: '#step-8',
    start: 'center center',
    end: '#step-8',
    pin: true,
    pinSpacing: false,
  });

  ScrollTrigger.create({
    trigger: '#turnout-stringency-chart',
    endTrigger: '#turnout-step-3',
    start: 'center center',
    end: '#turnout-step-3',
    pin: true,
    pinSpacing: false
  });

  ScrollTrigger.create({
      trigger: "#step-1",
      start: "top 90%",
      });

  ScrollTrigger.create({
      trigger: "#step-2",
      start: "top 80%",
      onEnter: step2,
      onLeaveBack: step2Back
      });

  ScrollTrigger.create({
      trigger: "#step-3",
      start: "top 80%",
      onEnter: step3,
      onLeaveBack: step3Back
      });

  ScrollTrigger.create({
      trigger: "#step-4",
      start: "top 80%",
      onEnter: step4,
      onLeaveBack: step4Back
      });

  ScrollTrigger.create({
      trigger: "#step-5",
      start: "top 80%",
      onEnter: step5,
      onLeaveBack: step5Back
      });

  ScrollTrigger.create({
      trigger: "#step-6",
      start: "top 80%",
      onEnter: step6,
      onLeaveBack: step6Back
      });

  ScrollTrigger.create({
      trigger: "#step-7",
      start: "top 80%",
      onEnter: step7,
      onLeaveBack: step7Back
      });

  ScrollTrigger.create({
      trigger: "#step-8",
      start: "top 80%",
      onEnter: step8,
      onLeaveBack: step8Back
      });

  ScrollTrigger.create({
      trigger: "#turnout-step-2",
      start: "top 80%",
      onEnter: turnoutStep2,
      onLeaveBack: turnoutStep2Back
      });

  ScrollTrigger.create({
      trigger: "#turnout-step-3",
      start: "top 80%",
      onEnter: turnoutStep3,
      onLeaveBack: turnoutStep3Back
      });
})

</script>
<svelte:window on:resize='{resize}'/>
<section class="intro">
  <img src="./ballots.jpg" alt="ballot" width="200">
  <h1> How COVID-19 disrupted elections around the world</h1>
  <a id="byline" href="https://twitter.com/mashfedorova" target="_blank">Maria Fedorova</a>
  <p class="intro-para">Between March 2020, when WHO has <a href='https://www.euro.who.int/en/health-topics/health-emergencies/coronavirus-covid-19/news/news/2020/3/who-announces-covid-19-outbreak-a-pandemic' target="_blank">declared</a> coronavirus (COVID-19) outbreak a global pandemic, and November 2021, 91 countries held national elections. Some of the initially scheduled elections were postponed primarily due to the challenges caused by the pandemic. Many countries held elections nevertheless. What characterized elections during these two challenging years? Below is an overview of all national elections that happened during the last two years.</p>
</section>
<div id="intro-chart" bind:this={width} >
   <IntroChart data={updatedData} width={widthChart} {cancelledBallots} {whichY} {whichX} {dataRectsPostponed} {monthsRaw} {regimes}
  constData={data} {dataPostponedBallots} {marginLeft} {colorBallot} {highlighApr} {legend} {opacityValue} {highlightLines}/>
  <!-- <CancelledBallots {cancelledBallots} width={widthChart}/> -->
  <!-- <ConnectingLine {data} {constData} width={widthChart}/> -->
</div>
<article class="scrolls">
  <section class="step" id="step-1">
    <p>Between March 2020 and November 2021, 19 elections had been <span class="postponed">postponed</span>, primarily because of the COVID-19 pandemic. </p>
    <p>The majority of elections were cancelled in the beginning of 2020, while only two countries - Iraq and Argentina - postponed their national elections in 2021.</p>
    <p>The chart shows all the elections at the time of when they were originally supposed to be held.</p>
    {#if width > 920}
    <p>One ballot<svg width="28" transform="scale(0.6)" class="ballot-inline"><path d="M23.3,31C16.2,31,9,30.8,1.8,30.2C1.2,20.7,1.3,11.2,2,1.7c7.1-0.1,14.2,0.3,21.2,1C24.1,12.1,24.1,21.6,23.3,31z M12.7,12.3c2.8-0.4,5.6-0.6,8.5-0.5 M13.5,21.3l7.2,0.2 M5.9,10.7l1.9,3.4l3.6-7.6" stroke="#000" fill="#f0e8e5" stroke-width="2.7"></path><circle cx="8" cy="23" r="3" stroke="#000" fill="#f0e8e5" stroke-width="2"></circle></svg>represents one national election (either presidential or parliamentary), held on a given day, and two ballots<svg width="29" transform="scale(0.6)" class="ballot-inline ballot-two"><path d="M23.3,28.1c-5.5,0.7-11.1,0.1-16.3-1.6c1-8.7,2.3-17,4-25c5.7-0.3,11.4,0.2,17,2.5C27.4,11.9,26.3,19.7,23.3,28.1z" stroke="#000" fill="#f0e8e5" stroke-width="2.5"></path><path d="M22.2,30.6c-6.3,1-12.7,1.8-19.1,2.3C1.6,24,1,15.1,1.4,6.2C7.3,5.6,13.1,5.1,19,4.9C19.8,13.4,20.5,21.9,22.2,30.6z M4.4,11.7c0.6,0.6,1.2,1.2,1.8,1.8c0.4-1.5,1.1-2.8,2-4 M11.6,11.4c1.1-0.4,2.3-0.6,3.6-0.6 M13.3,22.6c1.2-0.4,2.4-0.5,3.6-0.4" stroke="#000" fill="#f0e8e5" stroke-width="2.5"></path><circle cx="8" cy="23" r="3" stroke="#000" fill="#f0e8e5" stroke-width="2.5"></circle></svg> represent countries where citizens elected president and parliament on the same day. </p>
    <p class="para">Hover over ballots to see country names.</p>
    {/if}
  </section>
  <section class="step" id="step-2">
    <p>Most elections were not postponed for a relatively large amount of time. On average, elections were held two months after they were postponed.</p>
    <p>Here are those elections that were originally postponed shown on the month that they were actually held.</p>
  </section>
  <section class="step" id="step-3">
    <p>Between March 2020 and November 2021, elections were held in 91 countries. In total, 110 elections were held, with 18 countries having held multiple elections during this period.</p>
  </section>
  <section class="step" id="step-4">
    <p>In April 2020, a month after WHO declared the COVID-19 outbreak a pandemic many countries decided to cancel scheduled elections. Only two countries - Kiribati and South Korea - held elections nevertheless. </p>
    {#if width > 920}
    <p><strong>South Korea</strong> was widely <a href="https://aceproject.org/ero-en/misc/elections-and-covid-19-technical-paper-by" target="_blank">praised </a>for measures implemented during the elections. Early voting was encouraged, home voting provisions were extended, and stringent measures were implemented at the polling stations. Public approval of the government's handling of the pandemic was evident in a landslide victory of President Moon Jae-in’s governing party in parliamentary elections.</p>
    <p><strong>Kiribati</strong> went ahead with parliamentary elections, after having postponed them for a week because of disrupted public services. At the time of elections, however, there were no official cases of COVID-19 <a href='https://www.rnz.co.nz/international/pacific-news/414192/kiribati-goes-to-polls-as-normal-despite-covid-19-threat' target="_blank">recoded</a>.</p>
    {/if}
  </section>
  <section class="step" id="step-5">
    <p>Four countries - Chad, Haiti, Somalia, and Armenia - postponed elections and have not yet held these elections to the moment of publication.</p>
     {#if width > 920}
    <p> Some of these countries had more reasons to postpone elections than others. Armenia declared a state of emergency, having the highest number of COVID-19 cases among countries in the South Caucasus region. Parliamentary elections in Chad, initially scheduled in 2015, had been <a href="https://www.dw.com/en/chad-tensions-rise-over-debys-presidential-run/a-56536855" target="_blank">repeatedly</a> postponed even before the pandemic. President Idriss Deby had ruled Chad for the past 30 years, until his death in April 2021.</p>
    {/if}
  </section>
  <section class="step" id="step-6">
    <p>Elections were <span class="postponed">postponed</span>  in democratic regimes, as well as in authoritarian. However, twice as many democracies postponed elections as autocracies. While democracies always held postponed elections later, non-democratic regimes tended to postpone elections <span id='postponedGray'>indefinitely.</span></p>
  </section>
  <section class="step" id="step-7">
    <p>Non-democratic regimes tended to postpone elections for longer. While on average, <span class="demo">democracies</span> postponed elections for approximately three months, <span class="auto">autocracies</span> postponed elections on average for five months. and <span class="hyb">hybrid regimes</span> for four months.</p>
  </section>
  <section class="step" id="step-8">
    <p><strong>Ethiopia</strong> postponed parliamentary elections multiple times and for the longest period, for about ten months. Elections were originally scheduled for the end of August. At that time, 1,500 COVID-19 cases were recorded in the country, one of the highest numbers during the pandemic. The <a href="https://www.aljazeera.com/opinions/2021/6/18/why-ethiopias-elections-should-be-postponed" target="_blank">postponement</a> of elections, done unilaterally by the prime minister Abiy Ahmed was met with criticism by his opponents. </p>
  </section>
</article>
  <section class="textblocks">
    <h2>How electoral participation changed during the pandemic?</h2>
      <p class="para">
        With the pandemic affecting electoral calendars across the globe and political spectrum, has voter turnout<span class="tooltip">*<span class="tooltiptext">Turnout is defined as percentage of registered voters who participated in election</span></span> experienced any changes? External shocks could potentially influence voter turnout in two ways: (1) by increasing the cost of voting, leading to fewer people showing up at the voting booths, or (2) by having a mobilizing effect.
      </p>
      <p class="para">
        The analysis shows that, in general, turnout has declined slightly since the start of 2020 in comparison to elections held before. On average, turnout was 61%.
      </p>
      <p class="para">
         A few countries held elections with quite a low turnout, which was affected by many factors, not only related to the pandemic. One example is <strong>Algeria</strong>, where a referendum proposing constitutional changes was held in November 2020. Many <a href="https://www.aljazeera.com/news/2020/11/1/algeria-votes-on-referendum-aimed-at-ending-protest-movement" target="_blank">criticized</a> the referendum, pointing out that country's ultra presidential regime will not change. The opposition movement called for a boycott of the vote.
      </p>
      <p class="para">
        <strong>Egypt</strong> held <a href="https://www.economist.com/middle-east-and-africa/2020/09/03/egypt-looks-to-prosecute-millions-of-non-voters" target="_blank">elections</a> of the members of the Upper House that has no legislative powers, with only a  third of members being elected directly. While similar to the previous election, turnout was lower than during the elections held in 2011-2012, which were proclaimed to be the first democratic elections in the country.
      </p>
      <p class="para">
        <strong>Mexico</strong> held a referendum in 2021 on whether to investigate former leaders, amid criticism that the vote is a political stunt. Only 7% of registered voters casted their ballots.
      </p>
      <p class="para">
        <strong>The Dominican Republic</strong> held general elections in July 2020, one of the first countries to hold elections during the pandemic in the Americas. On the elections day, 1,036 COVID-19 cases were registered, with the cumulative highest number of cases up to that moment, resulting in a much lower turnout in comparison to the previous election.
      </p>
      <p class="para">
        Several elections attracted much more voters than previous elections, despite the pandemic. For example, <strong>Burundi</strong> held elections to replace President Pierre Nkurunziza, who had ruled the country for 15 years. A <a href="https://www.nytimes.com/2020/05/20/world/africa/burundi-election.html" target="_blank">competitive</a> race to elect a new president attracted  many voters, despite the widespread violence that accompanied the election.
      </p>
      <p class="para last">
        <strong>South Korea</strong> was one of the first countries to hold elections during the pandemic. The government's successful handling of the crisis brought it a landslide victory with the highest turnout since 1996.
      </p>
    <p class='turnout-title'>Voter turnout in <span class='demo'>democracies</span>, <span class='auto'>authoritarian</span>, and hybrid regimes</p>
    <div class="chart" bind:this={width}>
      <TurnoutChart data={dataTurnout} width={widthChart} x={xTurnout} xTicks = {xTicksTurn}/>
    </div>
    <p class='turnout-title'>Difference in turnout in comparison to the previous election</p>
    <div class="chart" bind:this={width} >
      <TurnoutChart data={dataTurnoutDiff} width={widthChart} x={xTurnoutDiff} xTicks={xTicksDiff}/>
    </div>
  </section>
  <section class="textblocks">
    <h2 class="title-turnout-strigency">Was turnout lower in elections with stricter lockdowns?</h2>
    <p class="para">Since the start of the pandemic, governments have implemented various measures aiming at curbing the virus. Covid-19 government response stringency <a href="https://www.bsg.ox.ac.uk/research/research-projects/covid-19-government-response-tracker" target="_blank">index</a> developed at Oxford University provides a comparable measure of severity of lockdowns across countries. </p>
    <p class="para last"> Did fewer people turnout out to the polling stations in countries where lockdowns were the most severe?</p>
  </section>
  <div class="chart" id="turnout-stringency-chart" bind:this={width}>
    <TurnoutStringency width={widthScatter} y={stringentChartY} {yTickLabel} {yHorisontal} {turnoutLabel} {rectTurnoutStringency} {regionsHighlight}/>
  </div>
    <article class="scrolls">
      <p class="step" id="turnout-step-1">Analysis shows that there was a negative relationship between turnout and stringency measures. In countries with more stringent policies, fewer voters showed at the polls compared to countries with lower stringency measures. </p>
      <section id="turnout-step-2" class="step" >
        <p >Turnout difference in comparison to the previous election has a negative but relatevely weak association with stringency measures. Approximately one third of all elections held between March 2020 and November 2021 had lower, or the same turnout as in the previous election and had a higher stringency index.
        </p>
        <p>Even though stringency measures are related to turnout, the weaker relationship between turnout difference and stringency policies might hint that other factors, such as how close an election is, might have a higher impact on turnout.</p>
      </section>
      <section class="step step-last-one" id="turnout-step-3" >
        <p>Elections during pandemic in <span id='africa'>Africa</span> tended to have higher turnout than in the previous election and lower stringency index than elections in other countries. Half of all elections with higher turnout and low stringency measures were in Africa. </p>
      </section>
    </article>
  <section class="textblocks">
    <h2 class="title-turnout-strigency">Did incumbents <span class="win">win</span> or <span class="lost">lose</span> more frequently?</h2>
    <p class="para">During the times of international crisises, citizens tend to unite behind their leaders and the support for the government is expected to increase. This a phenomenon known in political science literature as a "rally-round-the-flag" effect. </p>
    <p class="para">Overall, incumbents did win in more than a half of elections held during the pandemic. In democracies, however, the trend was more evident in 2020 when incumbents won a higher number of elections. In 2021, the number of elections with incumbents losing and winning was almost the same. </p>
     <p class="para last">Recent studies found "rally around the flag" effect in <a href="https://www.tandfonline.com/doi/full/10.1080/01402382.2021.1925017" target="_blank"> Austria, </a> and across <a href="https://www.pnas.org/content/pnas/early/2020/09/23/2009252117.full.pdf" target="_blank">11 different countries,</a> showing that political leaders got a boost in approval, especially in the early months of the pandemic. </p>
  </section>
  <div class="chart" bind:this={width}>
      <Incumbents data={dataIncumbents} width={widthChart}/>
    </div>
  <section class="textblocks">
    <p>.............................................</p>
    <p class="para">The pandemic disrupted the electoral calendar leading to many countries postponing elections. During the first months of the pandemic, political leaders got a boost in support demonstrated in many victories of incumbents. Half a year into the pandemic, however, elections got postponed much less frequently. Both democracies and autocracies postponed elections. While many countries opted not to hold elections during high infection rates, many went ahead despite high stringency measures. In those countries where stringency measures were particularly severe, fewer voters showed up at the polls.</p>
    <p class="para last">While different countries responded in various ways to the pandemic, there is no doubt that COVID-19 has had an enormous global effect on the ability of people to exercise their right to vote. New strains continue emerging, and no clear end to the disruption is insight. What will this mean for the long-term future of democracy?</p>
  </section>
  <section class="textblocks">
    <h3>Data sources</h3>
    <p class="para small">
      Data on election dates was collected using a compilation of information provided by <a href="https://www.idea.int/news-media/multimedia-reports/global-overview-covid-19-impact-elections" target="_blank">International IDEA website</a>.
    </p>
    <p class="para small">
      Data stringency measures:  Covid-19 government response stringency index developed at <a href="https://www.bsg.ox.ac.uk/research/research-projects/covid-19-government-response-tracker" target="_blank">Oxford University</a>.
    </p>
    <p class="para small">
      Data on regime types:  The Economist Intelligence Unit <a href="https://www.eiu.com/n/campaigns/democracy-index-2020/" target="_blank">Democracy Index 2020</a>.
    </p>
  </section>
  <section class="textblocks">
    <h3> Thanks</h3>
    <p class="para small small-last"> Special thanks to the invaluable comments of the members of Information Design and Elevate Groups, as well as Raúl & Lena!</p>
  </section>
<style>

.intro {
  margin: 0 auto;
  max-width: 650px;
  padding-top: 100px;
}

.intro-para {
  text-align: center;
  font-size: 1.2rem;
  padding-top: 40px;
}

img {
  display: block;
  margin-left: auto;
  margin-right: auto;
  padding-top: 100px;
}

h1 {
  text-align: center;
  margin-top: 70px;
  margin-bottom: 20px;
}

.textblocks {
  margin: 0 auto;
  max-width: 700px;
}

 h2 {
  text-align: left;
 }

.title-turnout-strigency {
  margin-top: 100px;
}

.last {
  margin-bottom: 95px;
}

.para {
  font-size: 1.2rem;
  line-height: 1.4;
}

.small {
  font-size: 0.9rem;
}

.small-last {
    padding-bottom: 50px;
}

#intro-chart {
  margin: 0 auto;
  max-width: 1200px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.chart {
  margin: 0 auto;
  max-width: 1200px;
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
  max-width: 600px;
  margin-bottom: 85vh;
  position: relative;
  background-color: #f0e8e5;
  padding: 20px;
  line-height: 1.4;
  font-size: 1.2rem;
  position: relative;
}

#step-1 {
  margin-top: -20vh;
}

.ballot-inline {
  max-height: 1em;
  display: inline;
  position: relative;
  vertical-align: top;
  overflow: visible;

}

.auto {
  background: rgba(242, 107, 107, 0.4);
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}

.demo {
  background: rgba(120, 89, 215, 0.3);
  background-size: 100% 90%;
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}

.hyb {
  background: #2bc48c54;
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}

#postponedGray {
  background: rgba(137, 132, 138, 0.3);
  background-repeat: no-repeat;
  background-size: 100% 90%;
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}

#africa {
  background: #22ab828a;
  background-repeat: no-repeat;
  background-size: 100% 90%;
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}

.lost {
  background: rgb(219, 189, 132, 0.3);
  background-repeat: no-repeat;
  background-size: 100% 90%;
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}

.win {
  background: #7099fa5d;
  background-repeat: no-repeat;
  background-size: 100% 90%;
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}


.postponed {
  background: rgb(112, 153, 250, 0.4);
  background-repeat: no-repeat;
  background-size: 100% 90%;
  background-position: 0 100%;
  padding-left: 2px;
  padding-right: 2px;
}


a {
 color: rgb(97, 94, 92)
}

.tooltip {
  position: relative;
  display: inline-block;
  color:rgba(112, 153, 250);
  cursor: pointer;
}


.tooltip .tooltiptext {
  visibility: hidden;
  width: 200px;
  background-color: #414141;
  color: #f0e8e5;
  text-align: center;
  border-radius: 6px;
  padding: 10px 10px;

  /* Position the tooltip */
  position: absolute;
  z-index: 1;
  bottom: 100%;
  left: 50%;
  margin-left: -125px;
}

.tooltiptext {
    font-size: 0.8em;
}
.tooltip:hover .tooltiptext {
  visibility: visible;
}

.turnout-title {
  position: relative;
  left: -20%;
  top: 50px;
  font-size: 1.2rem;
  font-weight: bold;
  fill: rgb(66, 66, 66);
}

#byline {
  text-align: center;
  margin: 0;
  display: block;
}


  @media only screen and (max-width: 1220px) {
    .turnout-title {
      position: relative;
      left: 0%;
      top: 50px;
      font-size: 1rem;
    }
  }


</style>