---
title: "Designing complex systems"
description: "Having worked across programming domains and technologies, I love building cross technology solutions that range across multiple platforms…"
date: "2020-05-09T11:16:16.186Z"
categories: []
published: false
---

Having worked across programming domains and technologies, I love building cross technology solutions that range across multiple platforms. An example of such a system is [http://slashux.io/](http://slashux.io/) which has a core desktop client written in C++, a photoshop plugin written with NodeJS, a front end panel written with AngularJS, a backend server written in Python and mobile client application for Android (Java) for the MVP.

Such complex systems are a lot of fun to build with small teams, SlashUX was developed by me and one other developer. With just 2 people involved in the project we could constantly iterate on the design specially in the initial version and work with a rapid pace. This allowed us to build the MVP from conception to a workable prototype within 3 months.

Building such a system can be very hard with bigger teams as it gets harder to be agile and do iterative development specially because it becomes slower to propagate concepts across the team. The easiest way to solve this problem is to design and architect the complete system before developing and document every aspect of it for others. This sounds great in theory but is almost impossible specially with massive systems that use cutting edge technology as there are too many variables involved for a ‘complete’ upfront design.

Recently at [Sense Health](http://www.sense-health.com/) I have been designing such a system that is currently being built by a team of 14+ engineers. The new solution ranges across multiple services, databases, languages and platforms. To ensure technically correct choices the system has to be agile so that experts from each platform can give input and design the system to be efficient, but to allow another engineer to design a sub-system matching with the complete solution requires them to have a complete understanding of how every component works and all design choices involved with it.

Explaining all this information can be very hard considering how some minor facets are modifiable by the engineers of the sub-systems too, to ensure technically correct choices.

I have been thinking about this problem set for some time and always find the definition of concepts in operating systems to be a great example of a successful design. Concepts in operating systems are not only supposed to be understood by all engineers in the development team but also by developers developing for the OS making them extremely complex. Concepts like Threads and Process’s were so well designed that they have survived the test of time across almost several major OSs.

When designing complex systems such concepts that are universal with almost no exceptions can be the **anchors** of the system. As the concepts have none or almost no exceptions, once understood they stay rigid and shape the design of rest of the system. When designing such concepts I prefer to use the term **promises**. 

Every anchor makes certain promises on the input receives, what actions it performs on the input and what resulting action can be produced. These anchors don’t have to be platform or service dependent and can be universal across the system. An example of such a promise for a thread would be that
