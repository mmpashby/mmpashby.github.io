---
layout: post
title: "Observability Series - What is Observability? - Logs"
date: 2024-06-28 12:00:00 +0100
categories: [tech, sre, observability]
tags: [tech, sre, observability]
---

Welcome to our exploration of observability! Whether you're a newcomer or a seasoned pro like myself looking for a refresher, you're in the right place. Grab a cup of tea, settle in, and let's dive into the realms of observability, site reliability engineering, alerting, and incident management. This mini-series will break down these complex topics into manageable parts, ensuring a smooth and informative read.

# Introduction

So, what exactly is observability engineering? Simply put, it's the ability to monitor, measure, and understand the state of a system or application through its external outputs.

Let’s use an analogy to illustrate. Consider a commercial aircraft. An aeronautical engineer relies on data from the aircraft to make informed maintenance decisions, such as monitoring the oil pressure of the jet engines. Pilots, on the other hand, need real-time metrics like altitude and cabin pressure to ensure safe flights. All this data is displayed on a dashboard, providing a comprehensive view of the aircraft’s performance. Manufacturers also analyse this data to plan improvements. To put it in perspective, a single commercial aircraft can generate up to 20 terabytes of data per engine, per hour of flight – a staggering amount of information!

In the same way, observability is crucial in modern software engineering. Mastering it means you can confidently answer how your system or application is performing without constantly checking if it's still running. Instead, you'll have a robust alerting and incident response platform, giving you peace of mind. After all, you want to make sure you know about an issue before your customers do!

As for my background, I’ve been in the observability field for years, working with network devices, server hardware, monolithic applications, micro-services, and event-driven architecture. I’ve designed and implemented platforms capable of handling tens of thousands of data points per second. My experience spans creating "observability in a box" solutions, offering platform-as-a-product services that software engineering teams can easily integrate with. I bring a wealth of knowledge and strong opinions on the subject.

# The Three Pillars of Observability - Logs

Warning, there be strong opionions here.

What are these three pillars of observability then? Great question, and this isn't a conclusion and term that I have coined, this was other super smart engineers in the observability space. I am only going to be covering logs in this post, because during the draft it was getting quite long and I want to make this a fairly digestable series.

## Pillar One - Logs/Event-Logs

Wait a minute, why Logs and/or Event-Logs? Whats the difference? Dont panic, we will get into that in a minute or two. Essentially, logs are human-readable flat text files that are used by engineers to capture useful data about their systems/services. Log messages occur when a developer deems it important to tell the system or application owner that something happened that they should probably know about. For example, your service could be dropping requests, and you should probably know why sooner rather than later.

![I got it!](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExZWIzN3l2YnZocm9uczZ0cjJuMTZlamR5czVob2h6emQydHJxMzdrZCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/HuI8ig8gydJjRbdC7E/giphy.gif)

Let's look at a good example of a log file:

```
[2021-02-23T13:26:23.505892 #22473]  INFO -- : [6459ffe1-ea53-4044-aaa3-bf902868f730] Started GET "/" for ::1 at 2021-02-23 13:26:23 -0800
```
*Source: The Path from Logs to Traces, by Alex Vondrak*

The example starts with a timestamp and a PID (Process ID) `[2021-02-23T13:26:23.505892 #22473]`, which is incredibly important for time-series investigation. If the logging entitys configured time is wrong, then this data is effectively useless. Precision timing is cruicial in modern software engineering. Please accept that as my first piece of sage advice - make sure you write **timezone aware code**, and the systems you host on are all connected to accurate centralised time protocol servers.

![Precision timing is so important](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExMmVjeWJ3ZHBpZmR4azR6ZjVoYXUzd2cybmt2ZThyam84czExbGN1NCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/gF5xLnIVPTs62OfVTJ/giphy.gif)

Next up we have the logging level, `INFO` in the example log file. Logging level basically means the "importance" of the log message to the system owner or operator. You may not care about INFO level log messages if your service is designed to accept tens of thousands of REST requests per minute. You may just want to collect metrics for each type of request that comes through. We will get to metrics on a later blog post. The logging level should **ALWAYS** be set at the application/service/runtime vars config stage, and the reason for that is that if you have to change it out in the wild for investigative purposes, ie. change to DEBUG level, you don't want to have to trawl through code in the early hours of the morning and update everywhere where the logging level is set.

![late night hacking](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExbGtremM0eXZoMTMzY3U3M2JjeHNqeWJoanE4MHNuZjhlanY2ZnF3ZyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/UqxVRm1IaaIGk/giphy.gif)

Next up we have something called a Uinvsersally Unique Identifier (uuid) v4 `6459ffe1-ea53-4044-aaa3-bf902868f730`, which is basically a randonly generated id to represent the generated request ID in this example. This request ID is important, to help chain events/units of work together for a given request.

We have a `GET` request next, which is one of the [HTTP verbs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods). It is useful to know the HTTP method used when making a request to a service/web app.

We then see a request path `/`. In this case, this is the root path of the application/service. We then see `::1` which is the host network address, an [IPv6](https://en.wikipedia.org/wiki/IPv6) localhost address. 

Finally, we have the requiest start time with the timezone UTC offset `2021-02-23 13:26:23 -0800`.

This was just purely a random example of how an application/service might log a message. There are many other examples out there where a lot of the formatting choices are relatively sensible out-of-the-box. I give another piece of advice to always find a good logging library for your chosen programming language, unless the standard library logger is really good already. You shouldn't want to reinvent the wheel for something like logging, just pick the most popular and easy to use open source solution. If you wan't to make modifications, then fork it, and create a new package, but always [inner source](https://en.wikipedia.org/wiki/Inner_source) it for your engineering team to encourage internal contributions.

![Teamwork!](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExdTA4NnYzc3hieHBoeHhxN3JvdWZodjBxMGJtd3E0ZGxweTdjeG5uayZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/dSetNZo2AJfptAk9hp/giphy.gif)

Now, some of the observant amoungst you may have remembered that we were going to discuss the difference between logs and event-logs. Don't Worry, I haven't forgotten! I also don't want to cover this topic in too much detail because otherwise the post will bloat quite significantly, but I will link you to some excellent views from the superstar of Observability Engineering, [Charity Majors](https://twitter.com/mipsytipsy). [Event-logs](https://charity.wtf/2019/02/05/logs-vs-structured-events/) are just better than logs. Thats right, to the annals of history with you logs! "But before we stand on that hill with you, Mark, can you tell us what the difference is, please?" Alright, I hear you. Event-logs are structured logs, and they follow a standardised format (JSON), which makes it trivial to parse and query for most centralised observability platforms. Let's take the log example from before and put that into an event-log:

```
{
  "name": "request",
  "timestamp": "2021-02-23T13:26:23.505892",
  "pid": 22743,
  "level": "info",
  "request_id": "6459ffe1-ea53-4044-aaa3-bf902868f730",
  "request.method": "GET",
  "request.path": "/",
  "request.ip": "::1",
  "request.start_at": "2021-02-23 13:26:23 -0800"
}
```
*Source: The Path from Logs to Traces, by Alex Vondrak*

I think you'll agree it's even easier to read in it's raw format. Lastly on event-logs - always emit a single event per request per service that it hits. Use the (Open Telemetry conventions)[https://opentelemetry.io/docs/specs/otel/logs/event-api/#event-data-model] and **ALWAYS** fire off an event before the request errors or exits the service, otherwise you have no breadcrumbs for your investigation! Also, if you have distributed systems, include a `trace_id` to pass onto other services in the stream.


## Wrap Up

Ohh, that isn't the last piece of advice regarding logs actually. My last piece of advice is to just avoid putting Personally Identifiable Information (PII) into your logs. You don't need it. If you have an anonomysed user id's in your data model, you **do not** need to log PII. Your security/compliance team will keep their hair, and you will also be way happier and content. But lets say you must absolutely include the event payloads from publisher event buses in your event-logs, then please please think about your event schemas. I will give a very basic example, but hopefully this highlights the point:

```
{
  "name": "request",
  "timestamp": "2021-02-23T13:26:23.505892",
  "pid": 22743,
  "level": "info",
  "request_id": "6459ffe1-ea53-4044-aaa3-bf902868f730",
  "request.method": "GET",
  "request.path": "/",
  "request.ip": "::1",
  "request.start_at": "2021-02-23 13:26:23 -0800"
  "request.payload": {
    "event": {
      "user.data" : {
        "id": "d6279b68-3460-4799-b26d-ea87a865f7fc"
        "private" {
          "full_name": "Joe Bloggs"
          "email": "joe.bloggs@example.com",
          "address": "1 Mount Olympus"
        }
      }
    }
  }
}
```

With the field `request.payload`, and the full field of `request.payload.event.user.data.private` I now know everything under that path is private user data, and I can filter/mask that at ingestion time. This is why engineering standardisation is so important in modern engineering orgs. You should agree with your fellow engineering community how to design your schemas to avoid problems later down the line, where mitigating PII data leaking into your observability platforms will be very difficult, and will force you down paths you will not want to go.

I hope I didn't miss anything important about logging in the world of observability, but if you feel I did, please tweet me and we can chat (my handle is linked on the left). Thanks for reading!
