---
title: "Building efficient micro services in Go"
description: "After dockerizing our existing infrastructure at Sense Health I have been working on building a suite of micro services for the backend. As…"
date: "2017-01-15T15:28:31.876Z"
categories: []
published: true
canonical_link: https://medium.com/@Rapchik/building-efficient-micro-services-in-go-61498e32b7f8
redirect_from:
  - /building-efficient-micro-services-in-go-61498e32b7f8
---

![](./asset-1.jpeg)

After [dockerizing our existing infrastructure](https://medium.com/@Rapchik/docker-in-development-339110a03c22) at [Sense Health](http://www.sense-health.com/) I have been working on building a suite of micro services for the backend. As with all micro services the core requirements are usually the same, fast communication, easy debugging and scalability.

To build this suite of micro services I made a lot of design choices and built a framework and collection of libraries which can be extended and would be used by all future services that we build in-house.

### Configuration

I use TOML files for all our configuration needs with [this Go package](https://github.com/BurntSushi/toml). TOML files are quite intuitive for configuration but more important the way the Go package does unmarshalling of TOML content from the file, it helps us have easily extendible configurations. With a single _toml.DecodeFile_ API call I am able to parse the entire config data structure including structures outside of my own package.

As the package uses reflection to find variable names in the TOML file and map it directly to the variable itself, we can have structures like:

> type Config struct {  
>  Name string  
>  Redis cachemgr.RedisConfig  
> }

Where the cachemgr is not even part of my project but the TOML package is able to fill the data structure from file making configuration much easier and less error prone.

### Communication (RPC, REST)

Communicating between services needs to be fast, reliable and built on a proven technology. For all inter service communication in our micro services I use [GRPC](http://www.grpc.io/) by Google. GRPC is designed on top of protocol buffers (a very efficient binary messaging framework) that allows you to do RPC calls across servers and comes with a ton of goodies (built in context & meta-data for authentication, HTTP/2 support, directional streaming support) and works in pretty much every language out there. GRPC is as good as it comes to have efficient, tightly coupled RPC calls between services.

We still have some REST end points in our services for communications with external services. All HTTP end points use Go’s built-in, net/http as the library of choice. I would strongly recommend using the built-in service if all your HTTP end points are dynamic and don’t involve serving static/cached content. For cached content there are some libraries like [fasthttp](https://github.com/valyala/fasthttp) which do in-memory caching and can be significantly faster.

For our use case as we only have dynamic content, I went for the internal net/http library as it supports HTTP/2 which fasthttp doesn’t for now. For routing the end points to their respective functions we used the [Gorilla Mux](https://github.com/gorilla/mux) which makes development process quite painless.

### Logging and Tracing

The collection of micro-services that we are building all work together and requests originating from one of our app can jump through several services before coming back with the final response. In such a case logging, profiling and error tracing is critical for the system to be stable.

I used [logrus](https://github.com/sirupsen/logrus) as the base of all our logging. With pretty coloured output on terminal, simple API calls and a much better way to work with object tracing then printf with parameters. Logrus also supports hooks to send log data to different services and works very well for all our logging needs.

With everything being dumped to log files, we are still only logging the service itself. With multiple services working together, we need more information than just a secluded log file of each service to trace problems. For a distributed tracing solution, I used [opentracing](http://opentracing.io/) with Z[ipkin](http://zipkin.io/) as the tracer backend.

Opentracing has support for multiple languages, is vendor neutral and has a solid community behind it making distributed tracing effortless. I attached hooks to all our GRPC and HTTP calls and made helper functions to make tracing and profiling functions more painless and automated during development.

These distributed logs are configured to keep data from only the last 2 weeks . I configured Zipkin to have a high sampling rate only during development. For our final production servers, we would have a low sampling count for Zipkin and a Logstash & Kibana server for performance metrics and ElastAlert or similar for error notification.

---

Overall I am very happy with this stack for Go and its allowing us to do very rapid development cycles, specially with distributed tracing and error logging we get enough flexibility to code aggressively and move fast as detecting slow and error prone code is as easy as opening the Zipkin web dashboard.
