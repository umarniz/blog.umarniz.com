---
title: "Thank you for the responses."
description: "My biggest complaint with most frameworks is that they try to provide the average minimum of everything that everyone needs. Polyglot…"
date: "2017-01-28T20:11:30.855Z"
categories: []
published: true
canonical_link: https://medium.com/@Rapchik/thank-you-for-the-responses-9ca7d051bcc
redirect_from:
  - /thank-you-for-the-responses-9ca7d051bcc
---

Thank you for the responses. I did consider go-kit and micro initially but chose not to go with an existing ‘complete’ framework of sorts for multiple reasons.

My biggest complaint with most frameworks is that they try to provide the average minimum of everything that everyone needs. Polyglot backends for transport/communication make little sense to me personally as I would prefer to have a single very efficient protocol for inter-services communications and a single user facing end point depending on the implementation of the service.

Right now we use HTTP/2 for end users and GRPC for backend communications and these two transports are not expected to be changed or replaced by any other service in the foreseeable future makes choosing a framework an ineffective choice as we just get more abstraction with no gains.

Having known transports also allows us to have very tight integrations with error handling (Zipkin automatically logs errors with payload without an explicit log call), logging/profiling (Logger/profiler automatically knows parent context in Zipkin with no need to explicitly pass it each call) and authentication (Built into the auth end-points which are different from all other end-points).

I think you guys are doing great work by providing an amazing tool kit that is ready to go out of the box and might be a great fit for a lot of people but considering I was looking for the bare essential libraries needed to build micro-services in Go that are tailored for my organisation, it was not the best fit for me.

PS: Thanks for the correction, I have updated Zipkin throughout the document :)
