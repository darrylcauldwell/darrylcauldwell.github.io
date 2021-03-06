+++
title = "Scrum Team Demystified"
date = "2014-08-26"
description = "Jira Overview - Terminology Demystified"
tags = [
    "devops"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
While I worked with Agile as a software tester when the concept first came out in 2002 shortly afterwards I joined a new company in a new role in Infrastructure, IT operations if you will. My recollection of Agile is that it was not a process at all; rather, it’s a set of principles summarized by the Agile Manifesto.

This seems like a lifetime ago,  but its finally made a comeback to my work life as I now work in team looking to foster a DevOps culture in our workplace.  A DevOps culture is an extension of Agile culture which rather than focusing purely on application life-cycle management it looks to now encompass server life-cycle management too. It aims to achieve this by breaking down organizational barriers to become a single cohesive team delivering a application service rather than discreet silo team structure.

![DevOps](/images/devops-scrum.png)

While Agile and DevOps are cultural changes and not methodologies in themselves there are methodologies which embody the principles such as Scrum, again there is no formal definition of a Scrum but industry members have formed frameworks such as [Mike Cohen](http://www.mountaingoatsoftware.com/agile/scrum). A scrum team is a strong informal collaboration of engineers and developers, within the loose role construct of a Scrum team.The team ideally are co-located, in same building, or at least same timezone so they can work closely and all be aware of each others work. The other ideal is that business or product owner while not sitting within the Scrum team is very close to them, and the team is given wide discretion to develop and innovate in order to reflect their informal understanding of the customer's real desire which is validated.

In support of this new way of working you can also use tools which support these frameworks.  One such tool is from Atlassian which includes Jira which is a tracker for team planning, it is used to capture and organize issues, assign work, and follow team activity. Atlassian provide an Agile plugin for Jira to give the schema extensions to give the console the concept of Sprint, Theme, Epic and Stories as a way to organize the issues and work, it also enables all of these things to be measured for velocity with regards to team performance.  At the side of Jira sits Atlassian Confluence essentially a Wiki for the creation of the evolving design documentation.

I mentioned above Sprint, Theme, Epic and Stories.  My loose definition of these terms stems from Mike Cohn's [Agile template](http://www.mountaingoatsoftware.com/agile). Where Scrum story is a user requirement definition,  a Scrum epic is a large user story, there is no threshold at where a story becomes a Epic it is just a big story,  potentially an Epic is a large story waiting to be more clearly defined and broken down into stories. A Scrum theme is a collection of Stories and \ or Epics. A Scrum sprint is a collection of Epics and \ or Stories which we hope to do within a set time frame. So we might have 100 stories pending,  and if our Sprint window is one week we might say this week we have these people engaged and can therefore complete these 10 stories. In practical terms for a sprint, we therefore draw a loose line in the sand that a Story should be achievable within a Sprint and an Epic a story which spans Sprints. Within Atlassian Jira story dependencies can be formed,  so if a Story hits an issue or hits a block the impact of this can be seen further up the dependency tree.

As well as managing the tasks and workload with Jira another key part is harnessing automation to give predictable and repeatable delivery of your application service.With the cloud comes the decoupling of operating systems from physical servers, storage and networking,  and with containers the further decoupling of application from operating systems. With self service software defined data center and OpenStack cloud management platform cloud style infrastructure deployments is becoming more accessible to the traditional private data centers too.  With so many configuration items required to deliver a application service within software defined infrastructure as well as the application itself.  It can be useful to harness desired state tooling such as [Chef](https://www.chef.io/chef/) and [Puppet](http://puppetlabs.com/), and leveraging distributed version control systems like [GitHub](https://github.com/), to manage, deploy and enforce your configuration.

A neat video on Jira, Agile, Scrum and Kanban

https://www.youtube.com/watch?v=NrHpXvDXVrw