---
layout: post
category : web micro log
---

**Notes on the MOOC "Enhance your career and Employability skills" Week 1** 

Integrity. "Form follows function" - results driven, challenging work. Flexibility with 
respect to tooling (flexibility/innovation over process). Data driven decisions. 
The customer is not always right; sometimes they don't know what they're looking for.
Mass collaboration over individual work. Automate relentlessly. Sense of purpose and results.

## Lifeline

* Positive Emotions: curiosity, hope, gratitude, joy, enthusiasm, pride, generosity  
* Negative Emotions: worry, dread, anger, sorrow, frustration, envy, selfishness  

<script src="http://code.shutterstock.com/rickshaw/vendor/d3.min.js"></script>
<script src="http://code.shutterstock.com/rickshaw/rickshaw.min.js"></script>

<div id="chart"></div>

<script>

var graph = new Rickshaw.Graph({
element: document.querySelector("#chart"),
width: 550,
height: 200,
renderer: 'line',
//each tick is a quarter, so 0 is the start of 2012, 1 is mar 2012...4 is end of 2012, 10 is mid of 2014 (this yr, i.e. now)
series: [{
data: [{ x: 0, y: 20 }, { x: 1, y: 17 }, { x: 2, y: 13 }, { x: 3, y: 11 }, { x: 4, y: 5 }, {x: 5, y: 9}, {x:6, y:15}, {x:7, y:5}, {x:8, y:17}, {x:9, y:15}, {x:10, y:13} ],
color: 'steelblue'
}, {
data: [ { x: 0, y: 10 }, { x: 1, y: 10 }, { x: 2, y: 10 }, { x: 3, y: 10 }, { x: 4, y: 10 }, {x: 5, y: 10}, {x:6, y:10}, {x:7, y:10}, {x:8, y:10}, {x:9, y:10}, {x:10, y:10} ],
color: 'lightblue'
}]
});

graph.render();
</script>

### Peaks

*  New Job  
*  Productivity  

### Troughs

*  Change in management  
*  Lack of acknowledgement/recognition  
*  Opinion and expertise is ignored  

## Test and Learn

"You don't know what you don't know"  

### Crafting Experiments

Currently I have the opportunity to place myself for nomination for the executive team for a local toastmasters club; I really should just see what it is like to be part of a committee.

Try doing tasks using different tools. Try generalising approaches. 

MOOCs

### Shifting Experiences

See how I could get a taste for leadership roles. Is it for me?


### Making Sense

_to be revisited_

## Value Grid

<table>
<tr><th>Value</th><th>Description </th><th>Free Choice</th><th>Half</th><th>Top three</th></tr>
<tr><td>Using your abilities</td><td>Not feeling like you could do the job with one hand tied behind your back. Stretching yourself. Using your skills.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Accomplishment</td><td>Feeling that you achieve something. You have clear goals. You can see a result for your efforts.</td><td>Y</td><td>Y</td><td></td></tr>
<tr><td>Being busy</td><td>Not having stretches of time when you have nothing to do. You have a buzz of activity.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Being responsible</td><td>Taking charge of your own work and the work of others. Being accountable.</td><td>Y</td><td>Y</td><td></td></tr>
<tr><td>Variety of task</td><td>Every day is not the same because you do different things.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Variety of environment</td><td>Every day is not the same because you are in different places.</td><td></td><td></td><td></td></tr>
<tr><td>Variety of contact</td><td>Every day is not the same because you are interacting with different people.</td><td></td><td></td><td></td></tr>
<tr><td>Adventure</td><td>You regularly take risks and have feelings of exhilaration or danger.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Fun</td><td>You are able to be light-hearted. You don’t have to be serious all the time.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Prestige environment</td><td>The place where you work is held in high regard as a major player in the field.</td><td>Y</td><td>Y</td><td></td></tr>
<tr><td>Advancement</td><td>There are opportunities to be promoted to higher positions.</td><td>Y</td><td>Y</td><td></td></tr>
<tr><td>Money</td><td>You earn or have the potential to earn a larger than average salary. You have perks, such as a company car, etc.</td><td></td><td></td><td></td></tr>
<tr><td>Development</td><td>There are opportunities to enhance, expand or develop your role and learn new things.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Recognition</td><td>When you perform well, your efforts are acknowledged or rewarded by praise, promotion or money.</td><td>Y</td><td>Y</td><td></td></tr>
<tr><td>Authority</td><td>You get to tell people what to do. You give direction to others.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Social status</td><td>You feel proud when you tell people what you do for a living. People think your job is interesting or glamorous</td><td></td><td></td><td></td></tr>
<tr><td>Colleagues</td><td>The people you work with are easy to get on with or interesting. There are opportunities for socialising with colleagues.</td><td></td><td></td><td></td></tr>
<tr><td>Helping individuals</td><td>You are involved in providing aid and assistance to people directly.</td><td></td><td></td><td></td></tr>
<tr><td>Helping society</td><td>You are doing something which contributes beneficially to society.</td><td></td><td></td><td></td></tr>
<tr><td>Caring</td><td>You are involved in showing support, empathy and love to others.</td><td></td><td></td><td></td></tr>
<tr><td>Nurturing</td><td>You are involved in helping other people to develop.</td><td></td><td></td><td></td></tr>
<tr><td>Justifiable</td><td>What you are doing fits in with your moral value system.</td><td></td><td></td><td></td></tr>
<tr><td>Fairness</td><td>You and other workers are treated fairly by your employer. You have good conditions of employment.</td><td></td><td></td><td></td></tr>
<tr><td>Spirituality</td><td>Your work allows you to express or explore your faith.</td><td></td><td></td><td></td></tr>
<tr><td>Management</td><td>You have a good working relationship with your boss. Your manager\'s way of working fits in well with your own.</td><td></td><td></td><td></td></tr>
<tr><td>Training</td><td>Structured opportunities for learning are provided and supported.</td><td></td><td></td><td></td></tr>
<tr><td>Creativity</td><td>You get to generate new ideas or solutions to problems. You get to innovate and be original.</td><td>Y</td><td>Y</td><td>Y</td></tr>
<tr><td>Decision making</td><td>You get to make some of the choices that affect your work.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Autonomy</td><td>You have some freedom to do things when you want and how you want.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Being expert</td><td>You have the opportunity to gain and use an in-depth knowledge of a subject. You are sought for advice in this area.</td><td>Y</td><td>Y</td><td>Y</td></tr>
<tr><td>Competition</td><td>You get the opportunity to test your abilities against others.</td><td>Y</td><td>Y</td><td>Y</td></tr>
<tr><td>Subject</td><td>Your work is enjoyable because you have a strong interest in the subject matter you are dealing with.</td><td></td><td></td><td></td></tr>
<tr><td>Aesthetics</td><td>You deal with ideas or things that are beautiful and require appreciation.</td><td></td><td></td><td></td></tr>
<tr><td>Quality</td><td>You work in situations in which precision and attention to detail are important, or where there is little room for error.</td><td></td><td></td><td></td></tr>
<tr><td>Security</td><td>You have security of employment. It is not likely that you will lose your job or have to find another job regularly.</td><td></td><td></td><td></td></tr>
<tr><td>Stress-free</td><td>You do not have to work under high levels of stress. The pressure of work is not too high and there is little conflict.</td><td></td><td></td><td></td></tr>
<tr><td>Health</td><td>Your work positively contributes to your physical and psychological wellbeing.</td><td></td><td></td><td></td></tr>
<tr><td>Stability</td><td>Work routines and duties are largely predictable and not likely to be subject to sudden or unforeseen changes.</td><td></td><td></td><td></td></tr>
<tr><td>Hours</td><td>You do not work more than average or irregular hours. Your job does not impinge on your family or social life. The patterns of work suit your lifestyle.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Resources</td><td>You have the materials, equipment and money you need to do the job. You are not expected to produce great results without the right tools.</td><td></td><td></td><td></td></tr>
<tr><td>Workspace</td><td>The place where you work is comfortable and suited to your working style and personal preferences.</td><td></td><td></td><td></td></tr>
<tr><td>Supportive</td><td>The organisation you work for is open and tolerant. Your views are sought and respected during decision making.</td><td>Y</td><td></td><td></td></tr>
<tr><td>Cooperation</td><td>Your work requires operating as part of a team and interacting with others to achieve a goal.</td><td>Y</td></tr>
</table>





