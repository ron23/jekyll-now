---
layout: post
title: Thoughts on architecture
---

#### Microservices / Serverless / Event-driven architecture

* retries: good & bad. Great to solve network failures etc. Bad in case on non-idempotent actions. 
In those cases, need to have some request ID to make sure the same request is not done multiple times.

* bottlenecks: might happen in the lambdas, or in the DB layer. Need to consider that and potentially throttle the queue.

* Serverless VPC issue: until recently, Lambdas that lived in VPC took long time to start. That was because they had to talk to DB in the VPC.
Couple "solutions" for that:

  * Warm up plugin: Have X amount of lambdas always warm by using a plugin that all it does it hit that lambda and exit.
That's ok, although ugly, but the problem is it doesn't scale so it completely misses the point.
If you warm up 50 lambdas, and you have 51 simultaneous requests, the last one is going to take a lot of time.

  * Have dedicated lambdas in the VPC that serve as a proxy. Other lambdas would be outside the VPC and will only talk to the lambdas in the VPC if it's actually needs the DB.
That's an ok solution but again, doesn't really "infinite scale". The lambdas in the vpc needs to be warmed up as well to respond quickly.
  * AWS came up with something that suppose to work fast. maybe this is solved.

* AWS best practice: use SNS + SQS. That allows sending the notification to multiple queus if needs. Also queue is more "robust" in the sense that it doesnt' drop messages.

* Logging and Tracing: maybe the most important thing acutally. Need to have some unique identifier per request that stays with the message all the way for easier tracing.
That allows to see exactly what happened per requests all the way.
CloudWatch is a bit problematic for that because it splits the lambda logs into multiple log groups, but it can be solved with either configurations or stream the logs to elasticSearch / Spark

* Local developemnt: Not very easy to do. Either you developer services independently and take care of integration on some complete dev environment or you run multiple services on your computer. 
Instead of mocking queue, you can use http and later switch to queueu and test on integration.

* Monorepo/multi-repo: Problem arise when you have some shared code, e.g "utilities".
  * If using mono-repo, all repos would be using the latest utilities, and to update you only need to update once.
  Tradeoff: You need to be damn sure you didn't break one of the microservices that depends on your utilities.
  It usually means have good testing suite and have some process for "breaking changes".
  
  * IF using multi-repo you can update the utilities and the other services won't have to use the updated version.
  It's easier to update utilities that way, but also problematic because now you have multiple versions of utilities running on your architecture which means bugs that you fixed might still be out there.
  Not easy to say which microservice uses which version etc. 
  
TBD...More to come







