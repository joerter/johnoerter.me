---
title: Bridge the Dev/Prod Gap with Docker
date: 17:34 07/09/2016
taxonomy:
    category: blog
    tag: [docker]
---

Presented at AIM HDC 2016 in Omaha, NE.

How many times have you implemented a feature in your development environment only to see it fail in production? This happens because our development, testing, and production environments often have significant differences that make shipping software less reliable. This talk will detail how to bridge this "gap" in environments with Docker.

Vagrant has helped us create reproducible development environments, but is hard to extend to production. Provisioning tools like Chef and Puppet can automate DevOps infrastructure, but aren't meant for development. Docker and containerization, however, allows us to duplicate our tools across environments to enable faster and more reliable deployments. We will learn about the best practices in achieving this goal, as well as the current challenges.

**Slides**: [https://speakerdeck.com/joerter/bridgedevproductiongapdocker](https://speakerdeck.com/joerter/bridgedevproductiongapdocker)
