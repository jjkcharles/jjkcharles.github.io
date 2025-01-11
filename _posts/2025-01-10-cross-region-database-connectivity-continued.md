---
layout: post
title: Cross-Region Database Connectivity Challenges - Part 2
subtitle: Spoiler, it works
author: jjk_charles
categories: cloud database architecture
tags: cloud-architecture
---

A while ago, I wrote about how I encountered a [performance issue with cross-region database connectivity]({% post_url 2024-03-22-cross-region-database-connectivity %}). At the end of that post, I mentioned that I had a potential solution in mind and planned to test it. However, the temporary solution we adopted worked well enough that we didn’t feel the need to revisit it. With other business priorities taking precedence, this issue was largely forgotten by the team.

Or, should I say, almost forgotten—because, over the holidays, I randomly started thinking about it again and decided to test the solution just to satisfy my own curiosity, if nothing else.

## A Recap of the Potential Solution
Here’s a quick reminder of the approach I was considering:

>1. Keep the database in the US region: Creating a regional replica would be cost-prohibitive.
>2. Deploy the APIs to EMEA/APAC regions.
>3. [Introduce PgBouncer]({% post_url 2024-01-26-central-connectionpool-pgbouncer %}): Use PgBouncer between the API and DB, hosted in the US region (closer to the DB).
>4. Persistent Connection Pooling: Configure PgBouncer with a pool of 20-30 connections to the DB and keep them persistent (min and max pool size set the same).
>5. API Connection to PgBouncer: API instances would connect to PgBouncer and maintain a persistent connection.

Instead of setting up an elaborate testing environment, I decided to simplify things. Since I'm not on my organization's Google Cloud instance, I had to figure out how to simulate the original problem of cross-region database connectivity. After a fair bit of searching, I settled on the following setup:

After some fair bit of searching for options, I landed on the below setup,

1. **A simple .NET console application**: This would connect to the database, run 50 queries, and log the time taken.
2. I’d repeat this for **different databases and configurations** (more on this below).
3. For the database to be hosted in different regions, I decided to use CockroachDB—they offer a free option, and it's wire-compatible with PostgreSQL.

## Testing Scenarios
In this round of testing, I wanted to evaluate the following configurations:

1. DB hosted on the same machine as the client application.
2. DB hosted in the GCP us-central1 region using CockroachDB, while the client application runs on my desktop.
3. DB hosted in the GCP eu-west1 region using CockroachDB, while the client application runs on my desktop.

For each of these configurations, I ran through the following scenarios:

1. No Connection Pooling
2. 1 Pooled Connection with a 5-second lifetime
3. 1 Pooled Connection with a 5-minute lifetime

I wrote a simple .NET console app that would issue 50 sequential queries to the database with a 0.5-second delay between each query.

## Results
Below are the performance results across the different configurations:

| Run Name | Average Time Taken (ms) |
|----------|-------------------------|
|Local - 1 Pooled Connection, 5 Minutes lifetime	| 1.84 |
|Local - 1 Pooled Connection, 5 Seconds lifetime	| 2.42 |
|Local - No Connection Pooling	| 7.7 |
|EU West - 1 Pooled Connection, 5 Minutes lifetime | 191.7 |
|EU West - 1 Pooled Connection, 5 Seconds lifetime	| 311.42 |
|EU West - No Connection Pooling	| 1167.48 |
|US Central - 1 Pooled Connection, 5 Minutes lifetime	| 101.24 |
|US Central - 1 Pooled Connection, 5 Seconds lifetime	| 156.78 |
|US Central - No Connection Pooling	| 623.96 |

As expected, the performance was poor when no connection pooling was applied. The performance improved as we pooled connections for a shorter time frame, with the best results seen when the connections were persistent (5 minutes in this case). Given the sample runs lasted about 1 minute, this persistence was sufficient for optimal performance.

>**Note**: I am based out of US West

## Visualizing the Results
Here are some charts that capture the timing across the different configurations and scenarios:
![Local](/assets/images/posts/cross-region-db-connectivity-continued-001.png)
![US Central](/assets/images/posts/cross-region-db-connectivity-continued-002.png)
![EU West](/assets/images/posts/cross-region-db-connectivity-continued-003.png)

>**Note**: The spikes on the red lines indicate connection disposal, which forces the application to reacquire a new connection to the DB, causing those performance dips.