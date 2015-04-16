---
layout: post
title:  "HPP: Express Middleware to Protect from HPP"
category: news
author: "ronfe"
---

[HPP](https://github.com/analog-nico/hpp ) is an express middleware to protect against HPP, namely HTTP Parameter Pollution attacks.

When receiving a reqeust, Express populates its parameters with the same name in an array.
That leaves potential attacking space for hackers. Attackers can intentionally pollute
request parameters to break the server down by simply repeat query parameters in request.
That is the basic form of HPP.

***

* HPP attacking is a sort of frequent, but not so 'advanced' attacking method. It could easily let the server down
* HPP middleware provides a tool to protect Express app against such attacking
* Seemingly, apps using RESTful APIs might not affected so much from HPP
