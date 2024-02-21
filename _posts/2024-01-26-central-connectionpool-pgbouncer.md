---
layout: post
title: Centralized Connection Pooling with PgBouncer
subtitle: Auto-scaling APIs vs. Connection Pooling
author: jjk_charles
categories: postgresql database sql
tags: pgbouncer postgresql connection-pooling database
---

[Connection pooling](https://en.wikipedia.org/wiki/Connection_pool) is often overlooked by developers, yet it is a crucial element in solving many application performance problems. I used to be in the same camp, not paying much attention when DBAs complained about one of the applications I was responsible for hogging too many connections and not releasing them soon enough after their use. This was many years ago - I've learned my lessons since then and have taken connection pooling seriously. However, a recent issue my team faced reignited my interest in this topic.

## The Context

My team develops and maintains a high-volume API serving tens of thousands of customers, generating hundreds of thousands of hits every day. Until recently, this was all hosted [on-prem](https://en.wikipedia.org/wiki/On-premises_software), but it was recently moved to the Cloud (GCP Cloud Run). While the transition itself was smooth (thanks to [Docker](https://www.docker.com/)), as bulky servers were replaced by small and rapidly scalable containers, we faced some challenges.

## The Problem

In the previous setup, we had two servers load-balanced to handle the load, each capable of handling approximately 1000 requests per second. The equivalent Cloud setup consisted of small instances capable of handling 50 requests per second, dynamically scaling up to 50 instances depending on load (for cost reasons).

Previously, each instance was set up to consume a maximum of 50 connections per database, which worked well. However, when migrated to the Cloud, this fact was overlooked. Consequently, our APIs, which used to make a total of 100 connections to each DB, ended up attempting to create up to 2500 connections per database. Since this DB was shared across multiple consumers, we had a hard limit of 500 simultaneous connections allowed to that DB.

> Even reducing the maximum connection pool size to 10 per instance per DB would have meant consuming all the connections the DB could support.

So, once that threshold is met:

1. This API was facing connection failures.
2. Other applications connecting to the database were refused further connections.
3. Overall database performance degraded due to the increased load on it.

While we could have simply increased the maximum connections allowed or created read replicas for the database to allow for more concurrency, these solutions would have merely thrown money at the problem without addressing the root cause.

### How did we solve this?

Fortunately, this issue manifested in a PostgreSQL instance. For PostgreSQL, we have a utility called [PgBouncer](https://www.pgbouncer.org/), which acts as a proxy for the PostgreSQL instance, effectively managing connection pooling.

We decided to introduce PgBouncer between the application (API) and Database, so the application doesn't have to directly interface with the Database. Since PgBouncer is wire-compatible with PostgreSQL, we didn't have to make any changes to the application code itself, except for config changes to point to PgBouncer.

PgBouncer supports three modes of operation:

1. Session pooling
2. Transaction pooling
3. Statement pooling

We opted to use "Transaction Pooling" with our application, which pools connections and assigns connections to `clients` on a `transaction` basis. Once the `client` completes a `Transaction`, the connection is returned to the pool to be reused for a different `Transaction`—either for the same `client` or a different one.

Here, you can control how many actual connections you want to establish at max with the Database. Since the connections are shared at the transaction level across all APIs, we wouldn't need as many connections as we originally needed when each instance was connecting to the database individually. We ended up configuring PgBouncer to use a maximum of 50 connections to the database (even fewer than what we had originally consumed).

After a few regression tests and performance tests, we were satisfied with the performance but noticed that our applications' use of `prepared statements` occasionally failed. This turned out to be unsupported in the version of PgBouncer we initially opted for (v. 1.16) at the `Transaction` level. We then had to upgrade our version to 1.21 and [configure PgBouncer to support it](https://www.pgbouncer.org/faq.html#how-to-use-prepared-statements-with-transaction-pooling).

Once this was set up, the API worked as expected.

While this is just scratching the surface of what is possible using PgBouncer, we are still figuring out ways to better host and share it across different applications, as well as gain visibility into which application is making the most connections, etc. Hoping to write more on this as I learn more about PgBouncer!
```