---
layout: post
title: "Power Up Asynchronous Tasks With Ec2 Spot Instances"
description: ""
category: 
tags: [ec2]
author: <a href="https://github.com/socrateslee">Chun Li</a>
---
{% include JB/setup %}

The [appflood.com](http://appflood.com) cross-promotion platform heavily adopts asynchronous task processing with the purpose of a very low web access latency. We deploy a distributed queue service using Amazon Ec2 spot instances to handle thousands of tasks per second.

You may bid the Ec2 spot instances with very very low price, usually, the price of spot instance are lower than reserved instances. Only the spots instances may be terminated anytime by Amazon. Let's see how using spot instances provides the cheapest solution for up and down scaling of our service.

+ __Realtime Monitoring__
We have a realtime monitoring of our distributed queue services and use monitoring data to determine whether to scale our service. When a peak of tasks comes, we shall scale up by opening more spot instances; when most of our worker process are spared, we shall scale down by closing spot instances with FIFO fashion.

+ __Prepare AMI for Fast Deploy__ 
Different from opening an on-demand instance, there is evalution time of you spot instance request before establishing a spot instance. So we prepare AMI to minimize the deploy time of new instances. 

+ __Regular Check of Instance State__
Since the spot instances may be terminated any time, we have a regular crontab job to check the states of all the spot instances. If any of spot instances is shutdown, and the status form realtime monitoring tell us the task is still fast growing, new spot instances will be deployed immediately.

+ __Redo of Failed Task__
Still, tasks may be interupted by the termination of spot instances. We track all of our tasks on worker processes, and regularly check if there are any running tasks on some ghost instances, if found, we will collect and reassign those tasks to other alive instances. 

+ __Dynamic Bidding Price__
The price of spot instances may fluctuation frequently. With the help boto library, we determine our bidding price based on the recent price history of our desired instance type. Note that Amazon wil only charge the spot instances with the current price, so it will not cost you much if you set a higher bidding price(Better for a spot instance stays from termination).


