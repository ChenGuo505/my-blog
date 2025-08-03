---
title: Debugging an AWS Security Group Leak in Crossplane-managed Kubernetes Resources
date: 2025-07-20 22:53:31
tags:
  - Kubernetes
  - AWS
  - Golang
categories:
  - Development
description:
  In this post, I’ll walk through how I debugged an AWS security group leak, the discovery process involving the AWS Load Balancer Controller, and how a subtle misconfiguration in our services led to the leak.
---

In one of my recent projects, I encountered a persistent and puzzling issue while managing AWS cloud resources via [Crossplane](https://crossplane.io/) in a Kubernetes cluster. Specifically, when deleting certain cloud infrastructure using Crossplane, the process would always hang — and upon investigation, the root cause turned out to be a leaked **security group** that was never deleted.

## The Problem

We were using Crossplane to declaratively manage cloud infrastructure (including networking, compute, and storage) within our Kubernetes cluster. However, during resource teardown — such as deleting a complete environment stack — we observed that deletion would consistently fail due to an **undeleted AWS security group**.

This wasn’t a random flake — it happened *every time*. A quick look at the leaked security group’s description revealed something interesting:

> *“Managed by AWS Load Balancer Controller”*

So the security group wasn’t created by us directly, but by the **AWS Load Balancer Controller (aws-lb-controller)** as part of its reconciliation logic. The problem was: when all related services were deleted, this security group still remained — blocking Crossplane from finishing its resource cleanup.

## Debugging

Naturally, I started with the basics: checking the logs, and searching through the [AWS Load Balancer Controller documentation](https://kubernetes-sigs.github.io/aws-load-balancer-controller/) and GitHub issues for similar reports.

Surprisingly, I couldn’t find any mention of this problem — no open issues, no discussions in the docs, no hints that this was expected behavior. This made the issue even more mysterious.

Digging into the controller’s source code, I found that it maintains an internal structure called sync.ObjectMap. This cache is used to track which Kubernetes services are currently associated with which AWS security groups.

Crucially, whether the controller deletes a security group depends on whether it *believes* the security group is still being used — and that belief comes from the **cached state**, not the live cluster state.

In our case, the controller was incorrectly thinking that the security group was still in use.

Eventually, the real issue surfaced: some of our Services, which were originally of type LoadBalancer, had been changed to ClusterIP as part of a recent project iteration. However, these Services *still retained annotations* like:

```yaml
service.beta.kubernetes.io/aws-load-balancer-type: "external"
```

These annotations are meant for the AWS LB Controller to act on LoadBalancer services. But because they remained on ClusterIP services, the controller was incorrectly trying to reconcile them as LoadBalancers — causing them to be added to the internal ObjectMap cache.

Then I double checked the logs and found the controller logged errors like:

```sh
unsupported resource type: ClusterIP
```

These logs confirmed my suspicion: the controller was attempting to process services it shouldn’t care about, failing halfway through reconciliation, and never cleaning them up from the cache. As a result, the associated security group was considered “in use” and thus never deleted.

## The Fix

Once I understood the root cause, the fix was simple: we removed the AWS LB-related annotations from all non-LoadBalancer services. After that, the controller stopped treating them as LoadBalancers, stopped adding them to the internal cache, and finally allowed the security group to be deleted as expected.

**Takeaways**

- **Caching can be dangerous**: Especially when the controller’s decision logic relies on an internal map rather than querying the live state.
- **Annotations are powerful — and risky**: Leaving legacy annotations in place can lead to subtle and unexpected side effects.
- **Logs are your friend**: The repetitive unsupported resource type: ClusterIP messages were the key clue that unlocked the whole investigation.
- **Controllers aren’t always idempotent or fail-safe**: This case showed how partial reconciliation can corrupt internal state if not carefully handled.

## Final Thoughts

Debugging this issue taught me a lot about the internals of AWS Load Balancer Controller and reminded me of the importance of thorough cleanups during service type transitions. Crossplane and Kubernetes controllers offer powerful abstractions, but with great power comes the need for precise configuration hygiene.

If you’re building infrastructure on Kubernetes and leveraging controllers like Crossplane or AWS LB Controller, always double-check annotations, reconcile logic, and watch for persistent logs. Sometimes the solution is hiding in plain sight.
