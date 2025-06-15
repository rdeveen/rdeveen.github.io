---
title: Using AWS SQS and Event Bridge Pipes to send messages to ECS
date: 2024-01-24 00:00:00 +1000
categories: AWS
description: "Learn how to use AWS SQS and EventBridge Pipes to send messages to ECS."
tagline: "Are AWS SQS Pipes reliably to send messages to ECS tasks?"
tags:
  - AWS
  - SQS
  - Event Bridge
  - Pipes
  - ECS
excerpt_separator: <!--more-->
header:
  overlay_image: ./assets/images/Angry-Blue-Tit-2025-Robert-de-Veen.jpg
  caption: "Photo credit: [**Robert de Veen**](https://www.robertdeveen.com)"
  show_overlay_excerpt: false
---

Today I was working on a integration to process message from an AWS Simple Queueing Services (SQS) Queue to a backend services. The specifics of this backend services doesn't really matter for this post, the only thing relevant is that it using the old-school Windows Communication Framework (WCF). With a custom .NET application written in the beloved C# language, we can communicate with the backend. 

This integration start with a third party putting a message on our queue. Because the amound of message we think this intergration would process we would like to make this integration serverless.