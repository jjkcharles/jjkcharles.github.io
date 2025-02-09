---
layout: post
title: Why Naming Things Right is Important
subtitle: A Simple Name Could Make All the Difference in the World
author: jjk_charles
categories: cloud general software
tags: cloud google-cloud-platform
---

In software programming, naming things clearly and accurately is crucial for both the developer who is building the software as well as the users who use the product or consumes the piece of code. A well-chosen name can reduce ambiguity, improve the overall user experience, and enhance the maintainability of code or services. It serves as an important tool for communication, ensuring everyone involved understands the functionality, behavior, and purpose of a given concept.

## A recent encounter

I have been using Google Cloud Platform (GCP) for around 4 years now, and I recently stumbled upon something that got me thinking about this topic.

GCP's Cloud Run service seems to have made a subtle but impactful change to its [billing configurations](https://cloud.google.com/run/docs/configuring/billing-settings). Previously, we had a config that was called “CPU only allocated during request processing” and “CPU always allocated”, to control whether we want the service to be always active listening for traffic, or wake up only in cases when traffic comes in.  While these names were functional, they could leave users scratching their heads, especially when it came to understanding their billing implications.

The new names, “request-based billing” and “instance-based billing,” bring immediate clarity. With these terms, the behavior is more intuitively understood. "Request-based billing" communicates that CPU resources are allocated only during active requests, while "instance-based billing" signifies that CPU is allocated for the entire lifespan of the instance, regardless of request activity.

This simple change in terminology makes it easier for users to quickly grasp how their billing is structured, without wading through technical jargon. It highlights how something as basic as naming can impact the user experience, making services more approachable and minimizing the learning curve.

## Takeaway
For developers and product teams, this is a reminder of the power of names. A well-thought-out name can provide a clearer mental model for users, reduce confusion, and improve overall satisfaction. When designing, building or documenting software, taking the time to get the names right is not just a good practice—it’s a fundamental part of delivering a great product.