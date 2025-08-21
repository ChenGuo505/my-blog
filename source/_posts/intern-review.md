---
title: Reflections on My Internship - Building, Securing, and Stabilizing Cloud Systems
date: 2025-08-21 22:57:56
tags:
  - Golang
  - Kubernetes
categories:
  - Development
description:
  A reflection on my three-month internship experience, covering Kubernetes controller development, cloud resource governance, and system stability improvements, along with lessons learned in balancing technology and operations.
---

Over the past three months, I’ve had the opportunity to work as a Cloud infra intern  at [PingCAP](https://www.pingcap.com/) that spanned **Kubernetes, cloud infrastructure, and system reliability**. It’s been a rewarding journey where I not only learned new technologies but also gained valuable insights into engineering practices in production systems. In this post, I’ll share what I worked on, the challenges I faced, and what I took away from the experience.

## Feature Development

One major part of my internship focused on feature development:

- **Cluster CR backup and query APIs**: I implemented periodic backups of cluster Custom Resource (CR) information and built query APIs, which were then integrated into the internal operations system. This involved using **Kubernetes, Gin, and GORM**.
- **TiDB Cloud dataplane manager**: I participated in the development of the Next-Gen control plane for TiDB Cloud, specifically contributing to the **dataplane manager**. My work centered on the **storage class and S3 bucket management module**, where I gained hands-on experience with **Kubernetes controller development, Crossplane, and AWS services**.
- **AWS load balancer subnet allocation design**: I proposed a new subnet allocation scheme for AWS load balancers to support TiProxy’s routing needs. Although we eventually discovered the original assumption was invalid (so the work didn’t proceed to implementation), the process taught me the importance of validating requirements early.

## Security Enhancements

Security was another recurring theme in my work:

- Defined **cloud resource naming conventions** and refined **IAM policies** based on them.
- Standardized **tag formats** across resources to support **cost analysis**.
- Added **deletion protection** for cloud resources via Crossplane CRs, preventing accidents caused by operator mistakes.
- Optimized **VPC networking** by cleaning up unnecessary route tables and security groups.

Through these tasks, I came to appreciate how security is not just about firewalls or permissions, but also about **clarity, consistency, and prevention of human errors**.

## System Stability Improvements

Reliability was a third area of focus:

- Strengthened testing by improving **unit tests** for the dataplane manager and scheduling **E2E tests** for regular execution.
- Enhanced observability with **Prometheus** for metrics collection, **Grafana** for visualization, and **alerting rules** for proactive monitoring.
- Fixed a critical bug where **security groups were leaked** during dataplane deletions, [see details](https://chenguo505.github.io/2025/07/21/alb-controller-debug/).

This work reminded me that stability comes not just from fixing bugs, but from building **processes and visibility** that prevent problems from happening silently.

## Growth and Reflections

Looking back, this internship helped me grow in three main ways:

1. **Technical depth** – I gained real-world experience with Kubernetes internals, controller development, and cloud-native tools like Crossplane and Prometheus.
2. **System-level thinking** – I learned to see beyond isolated tasks and understand how security, reliability, and cost all interact in cloud infrastructure.
3. **Critical questioning** – The AWS load balancer project reminded me that sometimes the most valuable outcome is realizing a problem doesn’t actually exist — and saving effort by not building unnecessary solutions.

## Closing Thoughts

These months taught me that working on cloud infrastructure is as much about **designing for people** (operators, developers, users) as it is about writing code. Good systems prevent mistakes, surface insights, and scale without drama.

I’m grateful for the guidance I received and excited to carry these lessons forward into future projects.
