---
title: Understanding Kubernetes Controller Development
date: 2025-06-04 22:33:16
tags:
  - Kubernetes
  - Container
  - Docker
  - Golang
categories:
  - Development

---

This post introduces the basics of Kubernetes controller development, covering key concepts, common patterns, and lessons learned from hands-on experience. A quick guide for anyone looking to extend Kubernetes with custom logic.
<!-- more -->

Kubernetes is not just a platform for container orchestration — it’s also a powerful extension framework. One of the most essential extensibility mechanisms in Kubernetes is the **controller**. Over the past few weeks, I’ve been diving deep into Kubernetes controller development. In this blog post, I’ll walk through the key patterns and share some personal insights I gained during the learning process.

## What is a Kubernetes Controller?

At its core, a Kubernetes controller is a control loop that watches the state of your cluster and makes changes to drive the current state toward the desired state. Most controllers are implemented using **Custom Resource Definitions (CRDs)** and **Custom Controllers**, allowing you to introduce entirely new APIs and behaviors into your cluster.

Kubernetes itself is full of controllers — from the Deployment controller to the Job controller — all managing resources behind the scenes. The beauty is: you can write your own.

## Core Concepts in Controller Development

1. **Custom Resource (CR) and Custom Resource Definition (CRD)**

   A CRD defines a new type of resource in the Kubernetes API. Once registered, you can create and manage instances of this new resource like any built-in Kubernetes object (e.g., Pods, Services).

2. **Reconciliation Loop**

   A controller continuously monitors CRs and reconciles the actual cluster state with the desired state declared by the user. The key function is the Reconcile() method, which embodies the control logic.

3. **Finalizers and Cleanup**

   Finalizers are essential when you need to perform cleanup operations before a resource is deleted. By adding a finalizer string to a resource, you ensure your controller gets a chance to do something (e.g., delete external resources) before Kubernetes removes the resource from etcd.

4. **Status Subresource**

   Status fields are used to reflect the current state of a resource. They help users understand whether the controller is still processing, or if something has failed.

5. **Controller Runtime (Kubebuilder)**

   Tools like [Kubebuilder](https://book.kubebuilder.io/) and [controller-runtime](https://pkg.go.dev/sigs.k8s.io/controller-runtime) help scaffold controllers quickly, handling informers, event handling, and reconciliation for you.

## Typical Controller Development Workflow

1. Scaffold a new API and controller using Kubebuilder:

   ```sh
   kubebuilder init --domain mydomain.io
   kubebuilder create api --group sample --version v1 --kind MyApp
   ```

2. Define the spec and status in your MyApp type.

3. Implement the Reconcile() method:
   - Fetch the resource.
   - Check current state.
   - Create/update/delete dependent resources (e.g., Pods, Services).
   - Update status if needed.
   - Requeue if the state is not yet desired.

4. Register your controller and run the manager.

## Personal Learnings and Reflections

1. **Reconciliation is Declarative, Not Imperative**

   One of the hardest mindset shifts was moving from an imperative programming model (do X, then Y, then Z) to a declarative one. In a controller, you don’t control the full execution path — instead, you respond to changes and let the control loop re-run until things converge.

2. **Statelessness and Idempotency Matter**

   A good Reconcile() function is **idempotent**, meaning it can be run multiple times without side effects. This is vital because the function might be triggered repeatedly for the same resource.

3. **Debugging Can Be Tricky**

   Since controllers are event-driven and asynchronous, bugs like race conditions, missed updates, or stale caches can be subtle. Adding detailed logs and using kubectl describe and kubectl get frequently helped me a lot.

4. **Finalizers Require Careful Handling**

   At first, I misunderstood finalizers — they are not just for deletion logic; they need to be explicitly removed from the resource once the cleanup is done. Forgetting to remove them leads to “stuck” resources that never get deleted.

5. **Tools Save Time**

   Kubebuilder abstracts a lot of boilerplate code. Without it, setting up informers, clients, and schemes can be tedious and error-prone. For production-level controllers, I highly recommend using it.

## Conclusion

Writing a Kubernetes controller is a great way to extend the platform in a cloud-native way. It teaches you how Kubernetes works internally, and how to build robust, scalable systems through declarative and reactive patterns.

If you’re comfortable with Go and want to build something Kubernetes-native, try writing your own controller. It’s a rewarding journey that deepens your understanding of the platform — and possibly gives your cluster superpowers.

**Resources:**

- [Kubebuilder Book](https://book.kubebuilder.io/)
- [Kubernetes API Concepts](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
- [Controller-runtime](https://pkg.go.dev/sigs.k8s.io/controller-runtime)

