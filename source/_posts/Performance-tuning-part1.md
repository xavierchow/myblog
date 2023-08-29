title: Performance tuning Part 1
date: 2023-08-29 23:14:29
tags: performance database
---

# Prelude
We received a critical incident report from Project X on the Production environment. The CPU of the Database (i.e. [Couchbase](https://www.couchbase.com/), more specifically, a bucket called members) increased drastically and reached 100%, and the system became unavailable for a few hours.

The lucky thing was that the system was restored after the traffic spike ceased, but the sad thing was ...

We didn’t know where the traffic was from and why it caused intensive CPU usage on Database, so the symptom may happen again in future.

 
<!-- more -->
# Attempts
An unknown root cause won’t block us from improvements; we are an efficient team after all. We carried out two plans according to some findings and assumptions.

Firstly, we checked the [N1QL](https://query-tutorial.couchbase.com/tutorial/#1) monitor dashboard, and found there was [named parameters](https://docs.couchbase.com/server/current/n1ql/n1ql-rest-api/exnamed.html) in some N1QL queries, which we reckon to be a slow query pattern, the improvement can be achieved by bumping up the version of the loopback-n1ql-mixin plugin with the [fix](https://github.com/Wiredcraft/loopback-n1ql-mixin/pull/53); what a low-hanging fruit! 

Secondly, you must heard, “Hardware is cheap, programmers are expensive.” If the Database is exhausted by CPU usage, let’s just give it more! Soon, the Database was upgraded from spec  4core 14G to  8 core 16G.

 
# Evaluation
We need a way to see whether the two aforementioned improvements work. The team prepared a performance test script with [k6](https://k6.io/). The script asesses two APIs on the backend component. (Don’t ask me why testing these two :-P )


```
# API-1
GET /api/internal/users/query?filter= { where: {
  or: [
  { 'oidc.sub': testUserFederationId },
  { 'email': testUserEmail},
  ]
}
```
```
# API-2
GET /api/admin/memberships
```
 

Surprisingly (should I?), there was barely any difference/improvement with the perf testing under 1k/2k/3k VUs, the CPU usage was still high; it just dropped from 75% to 70% even though we bumped 4vCPU to 8vCPU.

*note: VU means virtual user, see: https://k6.io/docs/get-started/running-k6/#adding-more-vus*
 

**What has been set wrong, then? **
 

# Revisiting
We were wrong, the mistake was not about the solution's effectiveness but about a tactical wrong direction. Due to some reasons, the team didn’t face up to the problem to find the root cause/bottleneck but chose to use a quick and no-brainer way, which eventually was a slow and inefficient.

> A body remains at rest, or in motion at a constant speed in a straight line, unless acted upon by a force 
> – Newton’s laws of motion

Inspired by Newton's law, we recalled that the CPU usage of DB was quite low and stable; something must be changed. What was the force then?


Thanks to the logging platform, with the help from the DevOps team, we cross-checked the timeline of the incident with the volume of requests, and identified a suspicious API was called heavily compared to the normal traffic(30% increase)
```
https://admin.some-system.xxx.com/api/internal/users/query?filter={
  "where":{
    "or":[{"email":"xxxx"},{"oidc.sub":"xxxx"}]
  }
}
```
This is the exact API tested by the performance test script,  calling it under 3K VUs did increase the CPU usage dramatically, so does it make sense that we say this API has performance issues and needs improvement?

 

No, we can not. 

Whenever we say a problem or an issue, it contains a few critical aspects, i.e., under a particular **Condition**, there is a **Gap** comparing the **Actual Situation** to the **Expectation**.
We know the increase of API calls is the trigger, and the high CPU usage is the result, but where is the expectation? Should that API be called so frequently? What is an appropriate rate,  will 100 RPS be good enough from the perspective of the caller? Or should that API be called by the caller at all?

 
All of those questions are homework for the team to answer; before we get the expected benchmark, it doesn’t hurt we should know the actual capability (current RPS) of this API for clarity.

Let’s continue with the next part, tuning with k6 to identify the RPS.

