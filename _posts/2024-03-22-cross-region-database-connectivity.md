---
layout: post
title: Cross-Region Database Connectivity Challenges
subtitle: Hmm...distance does matter
author: jjk_charles
categories: cloud database architecture
tags: cloud-architecture
---

At times, you run into challenges that push you to think creatively. Today was one of those days for me. Our DevOps team had been investigating a series of complaints from our EMEA and APAC customers about poor website performance. In hindsight, the issue was obvious: all our services were hosted in the US region, which caused significant latency for our users across the pond. No wonder we hadn’t heard similar complaints from NA or LATAM regions.

While the DevOps team worked hard to move most of the workload to their corresponding regions—mirroring services for both EMEA and APAC regions—there were still spikes in latency around key parts of the website after the migration. That’s when my team was asked to look into it.

We rely heavily on database queries in the areas my team manages, and it turns out that, for cost reasons, the databases weren't replicated in the corresponding regions. And now, you probably see where this is going…

##The Core Problem: Cross-Region API to DB Connectivity

While the API services were successfully migrated to EMEA and APAC regions, they still needed to reach the database in the US region. This cross-region DB connectivity introduced substantial latency, which became more problematic when the connection pool frequently disposed of DB connections. This meant that connections had to be reacquired often, which was both slow and costly.

>The reason we can’t have the API maintain long-lived connections to the DB is that the APIs run on 10-20 different Cloud Run instances at any given time, with the potential to scale up to 80 instances. If each instance were to maintain around 5 persistent connections, the number of active connections to the DB could quickly become unmanageable, leading to DB performance issues.

After much deliberation, we decided to move the APIs back to the US region. This would eliminate the DB latency, leaving us with only API-to-API calls that had to cross regions. Since these API-to-API calls were much less frequent than the API-to-DB calls, the overall cross-region latency was significantly reduced.

##Other Thoughts: A Potential Solution for the Future

While this solution provided immediate relief, I still had some ideas floating around for improving things long-term. We didn’t have the time to fully implement, test, and deploy a more permanent fix, but here's what I'm considering:

1. Keep the database in the US region: Creating a regional replica would be cost-prohibitive.
2. Deploy the APIs to EMEA/APAC regions.
3. [Introduce PgBouncer]({% post_url 2024-01-26-central-connectionpool-pgbouncer %}): Use PgBouncer between the API and DB, hosted in the US region (closer to the DB).
4. Persistent Connection Pooling: Configure PgBouncer with a pool of 20-30 connections to the DB and keep them persistent (min and max pool size set the same).
5. API Connection to PgBouncer: API instances will connect to PgBouncer and maintain a persistent connection.

The main idea is to establish the connection once and reuse it as much as possible. Introducing PgBouncer would help limit the number of connections the API holds, regardless of how many instances are running at any given time. This is not an ideal solution, but given the cost constraints on replicating the database, it might just be a workable solution.

Of course, this is still a theory at this stage, and I plan to run some experiments to validate it.