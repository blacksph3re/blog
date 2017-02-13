+++
date = "2016-10-10T20:00:00"
draft = false
tags = ["notifications", "messages", "microservices"]
title = "Notification pipelines"
math = false
+++

## Introduction
In this post I will share some thoughts upon implementing a notification pipeline in a microservice architecture. It should be said though that I don't yet have much practical experiences, this post is rather designed as a discussion about the topic. While working on the OMS for AEGEE, I found that this topic is actually quite a tough point when developing a distributed application.

* A lot of microservices will want to generate notifications for different things, resulting in a high number of notifications and a high diversity
* Users want to control how and which notifications they want
* Notifications need to be managed centrally, as it would get inconsistend having the notification settings everywhere
* Certain events could potentially trigger a high notification throughput (a huge event gets canceled, popular news post)
* There are several channels how notifications can be distributed: Mail, Pop-up notification, Push-notifications, Chat integration, ...

The good thing is that this topic is not necessarily time-critical. Noone will notice if the notification for an event creation is fired 30 seconds late, so it would be a good candidate for async execution and we don't need to be too careful on how long the job takes as soon as it is dispatched to a background task. Also, the notification object is quite simple, often just consisting of a recipient, message text, title, category and some information about the importance. The latter could be used to implement smart notifications like facebook, outputting a relatively constant number of pop-up notifications as long as the user is on the site but lowering the output when he leaves. 

## Design ideas
The first idea that came up when we discussed the topic **just an endpoint** on the service which also deals with user data. Just an API call that would immediately send an email to the user. It would be super-easy to implement and could potentially accept a big number of recipients. However we quickly noticed some drawbacks with this approach. It would be difficult adding or removing channels (mail/push/etc), as everything is concentrated in one endpoint. Also this could put some quite high load on the machine managing that endpoint - especially if the service is not replicable this could be problematic. However it would make it easy setting preferences, as the central management requirement would be fulfilled.

So we went on from this approach to a pub-sub idea, some kind of **observer-pattern**. We would still have the endpoint to send notifications, however we would also offer a hook for services to register with the core to get a copy of each notification sent by anyone. Those hooked microservices could be responsible for spreading the messages about one specific channel. This is a fairly easy to implement service and would keep the codebase small, ideal for newcomers in the project. However we still encountered a problem. We could not possibly maintain a centralized list of all possible notification categories. The services would grow constantly and would rapidly develop new classes of notifications, updating them into a centralized database would be a pain.

We came up with the idea of **tree-based categories**. There is a limited, predefined set of toplevel notifications, like "eventrelated", "userrelated", etc. This level is managed centrally and only updated rarely. Whenever a service wants to dispatch a notification, it has to fit it in one of the toplevel categories. However it also has the possibility to set an arbitrary amount of subtrees. This would allow us to firstly have very course-grained controls int he development process, but as the more finegrained category information is not lost, it would be easily possible to set up more finegrained controls later.

## Implementation
Well, right now there is no implementation, as this post is about ideas. I will update this as soon as we brought our ideas into practice. 