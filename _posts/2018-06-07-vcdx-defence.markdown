---
layout: post
title:  "VCDX6-DCV Defence Experience"
date:   2018-06-07 22:20:56 +0100
tags:
    - VMware
permalink: vcdx-defence/
---
In 2013 I was a systems engineer,  I worked mostly on automation scripting for consistently delivering virtualized infrastructure on VMware. A job I'd enjoyed but was ready to move my career forwards, my at the time colleague [Richard Dowling](https://twitter.com/virtRich) and I started looked into pursuing advanced VMware certification. In early 2014 we attended a VCDX workshop hosted by [John Arrasjid](https://twitter.com/vcdx001) at VMware EMEA HQ in Frimley. Also in the workshop was [Paul McSharry](https://twitter.com/pmcsharry) who John knew and who had recently had published [VCAP5-DCD Official Cert Guide](https://www.amazon.co.uk/VCAP5-DCD-Official-Cert-Guide-Certification/dp/078975018X).  I ordered the book that evening and this began my journey from engineer to architect.

During the four and a half years since I've continued with our goal of pursuing advanced VMware certification and passed VCAP5-DCA, VCAP5-DCD, VCAP5-DCV Design and VCIX6-NV. I'd also began contributeing much more to architecture design documentation but still had engineering duties. In 2016 I moved jobs to take on a role where I was a dedicated architect and co-authored a complex NSX for vSphere design with [Kyle McMaster](https://www.linkedin.com/in/kyle-mcmaster-a0995917/). I had every intention of submitting this for VCDX6-NV but on delivery of the project, I moved to work on Amazon Web Services and could not dedicate the time the VCDX process needed.

I changed job again to take another architect role, which aligned to business transformation program. Moving away from specific technology and closer to using various technologies as business enablers. As part of this I created a vSphere design for un-differentiated configuration, like a VVD but without vRA, smaller starting foootprint and tailed for our specific delivery org.

I speculative submitted this design exactly as was written for production with no tailoring for VCDX, and expected that it would be rejected but I would get good feedback on what needed to improve for a actual submission. From point of speculative submission I began writing a VCDX tailed document, after four weeks of working on new document set I got a surprise email reading, **"We are happy to inform you that you achieved a high enough score to allow you to proceed to the defence stage of the process."**

* Mistake One  - Never assume failure, and begin work not required. As with [Lean Architecture](http://darrylcauldwell.com/devops-architecture/) design only what is needed, enough but never more.

My acceptance letter arrived on Wedneday 9th May, defence day Wednesday 6th June, so exactly four weeks to prepare. Called out to the community, [Mat Jovanovic](https://twitter.com/matjovanovic) and [Bilal Ahmed VCDX-251](https://twitter.com/Dark_KnightUK) both pointed me to [Gregg Robertson VCDX-205](xhttps://twitter.com/GreggRobertson5) who runs the [VCDX Prep Slack Group](https://vcdxprepgroup.slack.com).

I'd made presentations of the design before to various people within a work context so had a lot of slides already. The community rallied and setup a mock defence for Monday 14th May. Albeit a short time , I had the whole weekend to refactor the slides... how hard could it be... oh it was my daughters 6th birthday on the Saturday and her party on Sunday. This is the point time became my greatest enemy, it all of course stemed from 'Mistake One'. The mock got completed, but I had planned to spend first 10-15m providing the C-level overview of the visio, conceptual design and then to logical. Then move on to questions, however as the mock panelists hadn't seen my design we somewhat deviated to a general technology walkthrough. My design is by its nature a re-usable design agnostic of a specific customer, my biggest takeaway was despite this I had to present from a customer context.

* Mistake Two - Even if design is customer agnositc, present in context of a customer story.

I spent the evenings of the next week, updating slide deck based on 1st mock. Had regular dialogue with [Bilal Ahmed VCDX-251](https://twitter.com/Dark_KnightUK) was was giving me mentorship, as my design was DCV it also included a chunk of NV so he put me in touch with [Shady ElMalatawey VCDX-249](https://twitter.com/ShadyMalatawey) and they both provided great mentorship.

* Tip One - Mentorship helps enorously, from the very basics like where to park grab luch etc, to the mental coaching and encouragement. Bilal was first person to say I was there on merit, and indeed the last person to remind me prior to entering room. There is a mental hurdle as well as technical, and personal coaching helps this side .. a lot.

* Tip Two - I am a naturally talkative guy, I love my design I have a lot invested in it, I naturally want to talk about technology when given chance. Bilal was very frank with me, and said basically **don't ramble**, aim for a 30s answer focussed on business aspects Recoverability, Aavailability, Managemability, Performance, Security and Cost. If they want more detail, they will ask,  answer succictly and move on.

The week 21-25th I arranged a mock defence with people internal to company I work for, I got six people to spend four hours and read design docs as submitted and held three mocks each with two panelists. I was very lucky to have lots of people who would spend time to help and so get very different feedback types, a three of deeply technical, a product manager, and two service architects. As well as answering their questions during session I noted these in an issues log, as I revised the presentation I closed out issues. Prior to defence I went back over the issues log as a whole to understand balance of areas people touched upon.

* Tip Three - Record all feedback and notes, its a great asset to review, conversations can be lost in the moment so having text reference help.

There was a 4hr [VCDX workshop](https://www.youtube.com/watch?v=Llty1D1yFII&t=1728s) on 25th, this is a relatively standard workshop which I had attended before. However the work I was doing meant I could listen in as I worked so I did. During this it became clear that a week after acceptance I should have had a 'Candidate Prep Call', this had never been scheduled. I mentioned on call and [Joe Silvagi](https://twitter.com/VMPrime) the workshop presenter setup a call directly after workshop (6pm Friday), I had personal engagement so spend a very few minutes on call where Joe made himself avbailable for any questions. I've no idea what is covered in a 'Candidate Prep Call' but would guess its useful.

* Tip Four - Make sure Candidate Prep Call gets scheduled within one week of acceptance, chase up if it doesn't.

During next two weeks I had a lot on at work, I held no more mocks, I barely managed to complete the slide deck up at 4am on day of defence updating.  I had no time to work at all on design workshop. This was a choice of course, I could have spent less time on defence and made sometime, I just couldn't see how this could be rehearsed effectively.

* Mistake Three - Don't be like Darryl, spend some time thinking and planning for design workshop

A few days prior to defence I had two emails, one with address to attend in Frimley and one with address to attend in Staines. I was pretty sure the Frimley office closed a year or two ago, but I never visit so couldn't be sure. Quick phone call to reception at Staines office confirmed I was booked and had parking space on site.

* Tip Five - Parking in Staines can be problematic, a call to reception got me an on-site space

The day of defence, I was up at 4am fussing about powerpoint, its a long drive for me so set off early and arrived two and half hours early for my 2pm slot. This time turned out to be very useful to reset the mind, had a walk by river, grabbed a leisurely lunch and forgot about the carnage of M25 traffic. VMware have great offices with large public spaces.

* Tip Six - Arrive early and reset the mind

The defence itself, nice well appointed room, four whiteboards each large, relatively new Windows laptop with USB-A ports only. I've been USB-C only for a while, but had guessed the supplied laptop might not be so had brought both USB-A and USB-C. On entering I was very nervous, the panelists could have taken advantage of this but let me get into a rhythm before asking questions. Joe Silvagi had said during VCDX workshop that the panelists asked questions to help you score points in areas they see weakness. This was really proved to be the case, the questions were asked in a way which seemed encouraging rather than confrontational. I made one possibly one possibly two assertions where they had asked a very technical question, I explained up to the point I was confident, then made caveate that next part I was relatively confident in but we should research the explicit detail around this. This is how I would approached with a customer and then followed up with detailed research, I expect same applies to this scenario. In hindsight this could be incorrect approach and better to speak up to point of explicit knowledge. I had mixed feelings about the backup slides through out, people had said they created but a waste of time, other people that they a great asset. I had created backup slides covering most areas of the design, the mock teams had said best layout for them a diagram on right and bullets on left. I spend a lot ... a very lot, of time creating these, in the end I used maybe 10-15 of them to help answer questions, if I was doing again I'd drop the text and have bigger pictures from backups.

* Tip Seven - Attend the vcdx workshop, listen to Joe

* Tip Eight - Pictures tell a thousand words, backup slides less text more diagrams, no pre-written text can cover the question

* Tip Nine - Pictures tell a thousand words but, presenting can be nervous having bulleted text can help you ensure you cover all the points you want to on that topic.

After defence and prior to design workshop I felt good, I love talking about my work, the team made me comfortable. I felt I gave a good account of design and business reasons behind, all in all it was a pleasant experience.

So the workshop, in VCDX workshop Joe had explicitly said you get to see the scenario slides, then when you start writting or ask first question the timer begins. As such I bearly listened as moderator read slides as I believed I had time to read at leisure, digest then, compose myself and then begin.  I had planned to create four large sections for Requiremnt, Assumption, Risk and Constraint,  and draw out the common parts of conceptual vSphere design while talking about establishing first requirement.

However, moderator stopped speaking then bang they started timer, this panicked me and I had a time limit to read which put me well and truely on the back foot from being relaxed. As such I forgot my planned approach and this became hinderance very soon as I wrote my requirements and constraints too close together and was hard to write, putting me further onto the back foot. It took me 5min or so to get back to where I wanted to start, and all because of incosistency in what I told prior and actions of moderator on the day (sigh). I never really got into a rhythm and just when I was starting to the moderator began laughing uncontrallably which made panelist laugh too, and me also, moderator left room to compose himself. This whole episode really made me loose my train of thought.

* Tip Ten - Attend the vcdx workshop, listen to Joe but don't expect the moderator to play by the rules

* Tip Elephen - Formulate a strategy and carry it out, don't allow yourself be distracted

On returning home late that night, my wife asked how did it go?  I said I enjoy talking about a design I'd invested so much in and which I was very proud of to a bunch of inteligent people.

She then asked do you think you'll pass? I thought about this, then said I've very little interest in the outcome. I set off on a journey four and a half years ago to transition to a new career, I now work in a team who design and engineer solutions which made a direct contribution to the success of my company. Obviously it would be the icing to the cake to get the certification, but whatever the outcome the vcdx journey itself has already given me all that I set out to achieve.

I've heard all leading up to this the failure rate extremely high, people even said close to 90%, so probability says that this my first attempt will be failure. So question is would I submit a 2nd, well I guess this is architecture so we could say what would ROI be, on the investment side it would cost $4000 + VAT in application fees, it would cost probably 30-40 hours of my time to prepare and a 10 hour day of defense. The interesting aspect now is what tangible return would I get over and above what I have today.
