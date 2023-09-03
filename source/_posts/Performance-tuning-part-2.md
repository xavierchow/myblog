title: Performance tuning Part 2
date: 2023-08-31 22:27:12
tags: performance database couchbase
---

> A good performance testing requires more than just executing a single script.

In [part 1](https://xavierchow.github.io/2023/08/29/Performance-tuning-part1/), I talked about the right way to identify and address the problem, now here I will talk more about what is a good performance testing and how to do a benchmark appropriately.

<!--more-->
# 5W1H works

* “Why” do we run the performance test? We may have different goals, you can refer to k6 doc [Understanding the different Types of Load Tests](https://k6.io/docs/test-types/load-test-types/#different-tests-for-different-goals) to decide the test scope and strategy separately.

* “What” to test? A specific backend component? A single API? or a bunch of APIs with a certain flow? What’s the parameter of the tested API? You'd better to have a clear target before you shoot the arrow.

* “Where” to run the test script? You may have multiple test environment, different environments may have different data sizes, which will impact the test result.

* “When?” You probably want to isolate the performance testing from the other testing scenario if multiple teams work on the same testing environment, also the performance test is better to be conducted after the functional test because a super performant but non-functional system does'nt make sense. 

* “Who?”,  in my company, sometimes the DevOps team, has the most convenience to run the test script across different environments. However, the Development team should drive the whole flow and evaluate the result with stakeholders(Product Owner, Architects, etc.) as they build the system and they know them best.

* “How?” Again it relies on the purpose of testing; basically, one thing we need to pay attention to is controlling the variables if we want to do a benchmark.

# Variables

In my case, I need to figure out the current RPS(requests per second) of the user query API. Again, keep in mind that the RPS is not an absolute constant number; it varies upon different conditions as follows,

| Factors |
|------|
| Server Resource(CPU, Memory, etc.) |
| Data size (Documents in the Couchbase Bucket, or records in the Table) |
| Network (Calling from external via CDN/Gateway vs Calling from internal) |
| API parameter/logic |
| Traffic / Pressure streched|

I planned to fix the all other factors, besides the Traffic / Pressure and use k6 options to vary the Traffic strategy.


# Key Concepts about k6
There are a few key concepts that you should understand to use k6.

***VU***:  Virtual Users are essentially parallel `while(true)` loops; generally, more virtual users mean more simulated traffic.

***Open & Closed model***:  In the closed model, VU iterations start only when the last iteration finishes. In the open model, on the other hand, VUs arrive independently of iteration completion, i.e., they don’t wait for the completion of iteration.

# Benchmark

In my case, as I aim to have a benchmark for our api, the [constant-arrival-rate](https://k6.io/docs/using-k6/scenarios/executors/constant-arrival-rate/) strategy is a better choice than VU-based strategies. 
Listing the metrics I got from the 1st round testing.

| Rate (iterations/sec) | CPU (%) | Latency (avg) | Latency (p90)|
|-----------------------|---------|---------------|--------------|
|  100 | 85% | 28.78s | 47.91s |
| 30 | 80% | 9.09s | 19.14s |
| 20 | 65% | 1.69s | 4.12s |
| 10 | 40% | 226ms | 464ms |
|  5 | 20% | 162ms | 331ms |

We can see two notable things here,

* The API is quite CPU consuming; 5 RPS introduces 20% CPU usage.
* Latency is not a linear increase along with the change of rate.

I have not made any improvement yet, however, instead of a random testing configuration like 1k/2k/3k VU, now I have a better inspection spot shows the sensitivity of change, that is, 20 iterations/sec, under which we could put change to other factors up on it(i.e., API parameters) to see the difference.
The 2nd round testing result as below, by tuning the API query,

|Rate |Query(filter)| CPU(%) | Latency(avg) | Latency(p90) |
|-----|---------------|--------|--------------|--------------|
|20 it/s | {"or":[{"email":"xxx@gmail.com"},{"oidc.sub":<br/>"b213acd3"}]} | 65% | 1.69s| 4.12s |
|20 it/s| {"email":"xxx@gmail.com"}')    | 20% | 62.33ms | 95.66ms|
|20 it/s | {"oidc.sub":<br/>"b213acd3"}       | 15% | 51.22ms | 77.93ms|
|20 it/s | {"or":[{"email":"xxx@gmail.com"},{"email":"yyy@gmail.com"}]} | 65% | 1.4s | 3.02s|


Highlighting some interesting findings here,

* single **where** criterion (either by email or oidc.sub ) is fast but still pretty CPU consuming.

* **or clause** is bad in terms of both CPU and latency.(even with two same pattern like email)


Next, we will deep dive into the N1QL world and use the benchmark above for root cause checking.

