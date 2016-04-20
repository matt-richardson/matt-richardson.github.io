---
layout: post
title: "New TeamCity Octopus Deploy Build Trigger Plugin"
date: 2016-04-20 07:47:34 +0100
comments: true
published: true
categories: ["blog", "archives"]
tags: ["On Software"]
---

My side project for the last few months has been a plugin for [TeamCity](https://jetbrains.com/teamcity) that allows 
you to setup build triggers for certain events in [Octopus Deploy](http://octopus.com).

As of today, I'm declaring it ready for public use. :D

It allows you to trigger builds when:
 - a deployment to an environment is complete (optionally only triggering on successful deployments)
 - a new release is created
 - a new tentacle is added to the server

This came out of some work I'm doing for a major UK fashion retailer in setting up TeamCity and Octopus Deploy and helping them move towards Continuous Delivery.

One of the tenants of the approach we used was that TeamCity was the orchestrator for everything. This has quite a few benefits, especially around using the TeamCity test runner and reporting capabilities for running tests against a deployed environment. Unfortunately, it also means that you need to duplicate all of your lifecycle, with a build configuration per environment for each project.

Out of this pain point, this plugin was born. It allows you to setup a single build that watches a specific Octopus Deploy project, and runs tests against the environment targeted when the project is deployed. TeamCity parameters are used to pass information about the environment, project and deployment triggered.

From here it was comparatively simple to extend it for the other trigger types. The ability to trigger when a release is created enables scenarios such as sending a notification to tell a team the release is available to be deployed.

Triggering when a new tentacle is added allows actions such as triggering a deployment of a given project (or projects) when a tentacle appears, such as in an auto-scaling environment.

If this sounds interesting to you, feel free to go [check it out](https://github.com/matt-richardson/teamcity-octopus-build-trigger-plugin). Issues, pull requests, and [general feedback](https://twitter.com/squire_matt) gratefully received.

## Reflection

Overall, its been an interesting foray into the land of Java - historically I've been focused on the Microsoft / .net space. Its been certainly challenging at times, having to learn IntelliJ IDEA, Ant, Maven, ngTest, Spring, JSP, and an awful lot more. Very glad I've challenged myself like this though.

Writing this plugin while reading [Release It!](https://pragprog.com/book/mnee/release-it) (by Michael T. Nygard) at the same time has been interesting as well, as its certainly impacted the way its written - especially around configuration, logging and analytics. If you haven't read it, make sure you do - its an invaluable resource for anyone in the DevOps / Site Reliability Engineer space.  
