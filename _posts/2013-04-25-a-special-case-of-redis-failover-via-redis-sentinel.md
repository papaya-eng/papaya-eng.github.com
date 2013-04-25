---
layout: post
title: "A Special Case of Redis Failover Via Redis Sentinel"
description: ""
category: 
tags: [nosql, redis]
author: <a href="https://github.com/socrateslee">Chun Li</a>
---
{% include JB/setup %}

Redis sentinel is a great tool for autonomoous failover of redis servers. The detail of how to setup and use redis sentinel is available in [the redis official documents](http://redis.io/topics/sentinel).

What we gonna talk about here is a special case. First let's assuming we have following quick setup(all use redis 2.6.12):

    # redis master on port 6400
	redis-server --port 6400
    # redis slave on port 6401
    redis-server --port 6401 --slaveof 127.0.0.1 6400
    # redis sentinel
    redis-server sentinel.conf --sentinel

The sentinel.conf is

    port 26379
    sentinel monitor mymaster 127.0.0.1 6400 1
    sentinel down-after-milliseconds mymaster 5000
    sentinel failover-timeout mymaster 900000
    sentinel can-failover mymaster yes
    sentinel parallel-syncs mymaster 1

So the if the redis master on port 6400 crashes, the sentinel will promote the redis slave on port 6401 to be the new master, everything goes as expected. The special case is, since the sentinel.conf is not yet changed(still monitors the old redis master), if the redis sentinel process is restarted at this moment, what will happened?

Let's see how the redis sentinel process acts in 3 conditions.


First, if the old redis master(on port 6400) is already resurrected and demoted as the slave of the new redis master(on port 6401). At this moment, if we restart the redis sentinel, we shall get the message like this

    [15040] 25 Apr 09:35:40.786 # +redirect-to-master mymaster 127.0.0.1 6400 127.0.0.1 6401
    [15040] 25 Apr 09:35:50.867 * +slave slave 127.0.0.1:6400 127.0.0.1 6400 @ mymaster 127.0.0.1 6401

And the redis sentinel will have the right redis master to monitor, the old redis master is also correctly in the slave list.

Second, if the old master(on port 6400) still in the down state. After restarting the redis sentinel, we shall see that the sentinel procdess are monitoring the old master, and will consider it objectice down(ODOWN), like

    [15040] 25 Apr 09:32:03.817 # +sdown master mymaster 127.0.0.1 6400
    [15040] 25 Apr 09:32:03.819 # +odown master mymaster 127.0.0.1 6400 #quorum 1/1

And if restart the old master as the slave of the new master(on port 6401), the sentinel will follow the case of the first condition, redirecting the monitoring to the new master.


Third, if the old master(on port 6400) resurrects and be slave of no one. When the redis sentinel process restarts, if will simply monitor the old master and no slave is found. But if you use the redis client send a

    slaveof 127.0.0.1 6401

The sentinel will not have a redirect-to-master message, and it still monitors the old master. The redis sentinel process will redirect to master the new master(on port 6401) if you restart the sentinel process or reset the master via a redis client by

    sentinel reset mymaster

The sentinel will have the following message:

    [15093] 25 Apr 09:46:33.046 # +reset-master master mymaster 127.0.0.1 6400
    [15093] 25 Apr 09:46:36.071 # +redirect-to-master mymaster 127.0.0.1 6400 127.0.0.1 6401
    [15093] 25 Apr 09:46:46.152 * +slave slave 127.0.0.1:6400 127.0.0.1 6400 @ mymaster 127.0.0.1 6401


So this the case that may occured, when the redis sentinel process restarts and how it activated with the existed configuration file(if you haven't change it). In document of the redis sentinel, the situation of the master processes resurrect will be handled in 2.6.13, and you should be able to easily configure the old master to be the slave of new master when resurrected.

In the practice, you should have multiple redis sentinels running. If the configuration files of every sentinel process is not consistent, the redis sentinel process may fall apart into seperated groups when several of the prcesses restarts. We should be able to use service like zookeeper to maintain the consistentcy of configuration files.  
