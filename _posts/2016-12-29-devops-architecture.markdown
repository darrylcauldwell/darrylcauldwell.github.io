---
layout: post
title:  "The DevOps Infrastructure Architecture"
date:   2016-12-29 11:20:56 +0100
tags:
    - DevOps
permalink: devops-architecture/
---
One common critisism of Agile teams is they are architecturally weak and disconnected from the operational realities of complex enterprise environments. Traditional enterprise architecture processes are viewed focusing on delivering documentation rather than delivering a working solution, slow, and focussed on delivering monolith design rather than continuous delivery of small incremental updates. 

Traditional Infrastructure Lifecycle
------------------------------------
Traditional infrastructure architecture development metholodies are a series of cyclical processes which culminate in a set of detailed documentation, including conceptual design, logical design and physical design. These are developed by an architect or architectural team in relative isolation to day-to-day operational activities. Typically in a traditional large enterprise architectural disciplines are aligned to technoligies wintel, security, network, storage, & *nix and each prepares there own documentation set. 

At the point of delivery it is often found that the documentation sets are so large that they are often delivered incomplete of the full detail needed to deliver the solution. Cross discipline highly skilled engineers with deep technical product knowledge then engineer the solution design to make the component pieces work together. The engineers then delivers automation scripts to deliver the exact settings needed for the solution to operate. At this point the carefully prepared architectural documentation set is out of date.

Enterprise solutions are usually intended to plugin to existing operational tooling, and the enterprise architect usually assume this tooling will have suitable functionality built in to support the solution they design. Once deployed it is often found that the integration to operational tooling works at a very basic level but does not out of the box meet the management complexities of the solution delivered. At this point cross discipline highly skilled engineers with deep technical product knowledge have to get together with the operational tooling team and engineer a solution to make the solution work. These changes to the solution make the architectural documentation further out of date. Day two operations of any delivered solution will naturally lead to more change and the document set becoming more obsolete. All of these changes require high skilled people at every stage of delivery.

Agile Infrastructure Architecture
---------------------------------
Pre-virtualisation the requirement for up front design was partially justified as the hardware required for the application needed to be specced, procured, delivered, racked and powered before it could be consumed. Virtualization de-coupled these two processes and pools of compute resources could be pre-provisioned and consumed elastically, the public cloud provides further elasticity in delivery.

Agile development techniques such as ExtremeProgramming revolve around 'You Aren't Going to Need It (YAGNI)'. This principle relates to the notion of 'incremental design' rather than carefully considered planned development encompassing all possible future need. In lean this is refered to as just-in-time. Agile architecture should be a collaborative activity which enables agile development to progress features and improvements in a timely manner to produce artifacts to meet the immediate requirement, and iterated just-in-time. The use of architecture model storming can be used between, architecture and operational engineering teams to agree solutions collaboratively.

The agile infrastructure architect still has the primary goal of designing solutions which are deployable, scalable, secure, maintainable, managable, standards compliant, fault tolerant, testable, upgradable and recoverable.

When architectcure design is designed and delivered in small units, the feedback loop to deploying engineer is much shorter, issues are found and fixed and all stakeholders are kept in the loop. The use of collaboration tools to store documentation such as Github WiKi's or Atlassian Confluence means that documention can be kept up to date collabortaively and be kept close to the automation scipts. 

References
----------
[Agile Vs Kanban: What’s the Difference?](https://www.guru99.com/agile-vs-kanban.html)
[Agile Architect: Role](http://www.agilearchitect.org/agile/role.htm)  
[Martin Fowler: YAGNI](http://martinfowler.com/bliki/Yagni.html)  
[Agile Modeling: Agile Architecture](http://agilemodeling.com/essays/agileArchitecture.htm)  
[Agile Modeling](http://www.agilemodeling.com/)  
[What Every Successful Product Manager (Owner) Should Know](https://www.gilb.com/blog/what-every-successful-project-manager-should-know)  
[Lean Architecture: Just In Time, Just Enough](http://blog.xebia.com/lean-architecture-principle-5-just-in-time-just-enough/)  
[Extreme Programming: A Gentle Introduction](http://www.extremeprogramming.org/)  
[Scaled Agile: Agile Architectuere Abstract](http://www.scaledagileframework.com/agile-architecture/)  
[Martin Fowler: Is Design Dead](http://martinfowler.com/articles/designDead.html)  
[Agile Modeling: Model Storming](http://agilemodeling.com/essays/modelStorming.htm)  
[Shaping Software: Agile Architecture Method](http://shapingsoftware.com/2009/03/02/agile-architecture-method/)  
[Agile Data: Agile Enterprise Architecture Approach](www.agiledata.org/essays/enterpriseArchitecture.html#AgileApproach)