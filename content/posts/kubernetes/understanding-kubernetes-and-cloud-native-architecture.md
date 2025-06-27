+++
title = 'Understanding Kubernetes and Cloud Native Architecture'
date = 2025-06-24T17:00:04+01:00
draft = true
image = '/images/kubernetes-101-banner2.png' 
tags = ['devops', 'kubernetes']
+++

Like many backend engineers, I have spent most of my time doing CRUD, writing APIs, database queries, engaging in pointless arguments about programming languages and so on. But I began to notice that most job descriptions for senior roles expected more than just programming and database skills.

Since the rise of cloud computing, understanding infrastructure management, DevOps skills and cloud-native experience are now becoming essential for a backend engineer.

This realization led me to take a meaningful step forward in my professional journey, learning [Kubernetes](https://kubernetes.io). I came across a [CNCF](https://cncf.io) and [Andela](https://andela.com) [partnership](https://www.andela.com/blog-posts/andela-and-cncf-team-up-to-give-the-11-8b-kubernetes-market-30-000-new-players) offering to train developers in kubernetes which I did not let pass me by.
I got accepted into the first cohort of the program, took the [LFS250 course](https://training.linuxfoundation.org/training/kubernetes-and-cloud-native-essentials-lfs250/) and within four weeks obtained my [KCNA](https://training.linuxfoundation.org/certification/kubernetes-cloud-native-associate/) certification as a kubernetes noob.

This post is a reflection of what I learned, why it matters, and where I'm headed next.


## What Even Is Cloud Native

Before diving into kubernetes, I want to talk about cloud-native architecture. Cloud-native is more than just running applications in the cloud, but it is a set of technological and architectural principles and practices geared towards cost-efficiency, reliability and faster time-to-market.

Some of the core principles or philosophy of the cloud native architecture include:

- Containers as a unit of deployment.
- Loosely coupled systems with microservices as the default design pattern.
- Employing automation and CI/CD pipelines for fast, repeatable and reliable delivery of applications.
- Dynamic orchestration of infrastructure using tools like kubernetes.

The cloud native ecosystem is vast with all kinds of tools for monitoring, networking, storage and so on, but kubernetes is at the center of it all. You can read more on cloud native [here](https://github.com/cncf/toc/blob/main/DEFINITION.md).


## What Is Kubernetes
Going by current trends, Kubernetes can be considered as the platform for deploying and managing modern applications. 
Kubernetes is a [container orchestration]() platform that automates the deployment, scaling, networking and self-healing of containerized applications.

In practice, Kubernetes takes your containerized workloads and ensures they're always running in the desired state, no matter what. If an application crashes, Kubernetes restarts it. If traffic spikes, it can add replicas to manage load. If a node dies, it reschedules your workloads to healthy nodes.

Containers are a very convenient and lightweight way to run applications for a lot of good reasons.

Before containers you had to run applications on virtual machines or physical servers, running more than one application on the same server would require that these two do not clash or interfere with each other in terms of resource usage for both software and hardware. 

Consider a case where two different applications both need different versions of a system library, or one application consumes much of the available resources and cuases all other process to crashes, this is not ideal and we have to ensure that each application is isolated from the other.

Virtual machines allow you to isolate operating systems on the same hardware, so applications running in virtual machines are isolated and resource limits and constraints can be well defined.

The success of virtualization technology was the ability to run multiple operating systems while using same hardware, but the down side is it the operating system overhead makes it somewhat inefficient. 

During restarts, you have to add boot times into consideration, the storage space used to install full operating systems on disk and as different applications have different resource requirements, it would be wasteful to run a static website in a virtual machine, compared to an e-commerce website.

Containers are like virtual machines in the sense they isolate processes, but unlike virtual machines, containers share the same underlying host operating system kernel. This makes them very lightweight and cheap to run.

Many organizations nowadays have multiple services, or multiple instances of the same service to spread load across these instances. Managing tens of containers manually is easy, but managing containers at scale gets very messy easily and we will see why.
When you deploy a containerized application, you need to setup networking, manage restarts when it crashes, update to new versions and scale your instances when demands are high.

Container orchestration is a solution to this problem at scale and kubernetes is a container orchestration system. It can help manage automatic deployments, scaling, fault tolerance and management of containerized applications.

## Kubernetes Architecture

Kubernetes operates in a cluster which usually spans across multiple hosts or nodes. The cluster can be divided into two parts, the control plane  and the worker plane. 
The control plane is the brain of the cluster, it manages container scheduling/placement to worker nodes, self-healing, deployments etc. The worker nodes run the containerized workloads.

Some of the major concepts in kubernetes are as follows:

- Pods: the smallest deployable units, often one container, sometimes more.

- Deployments: manage the lifecycle of pods.

- Services: expose your pods internally or externally.

- Nodes & Clusters: infrastructure components that run and manage workloads.

- Control Plane vs Worker Nodes: the brain and the muscle of the cluster.


Kubernetes is an absolute beast and is one of the most complex tools to learn in software engineering. One blog post cannot be enough to capture the depth of kubernetes. So I am going to start simple by building 10 simple projects that highlights the different parts of kubernetes.

I started this journey to grow beyond my backend bubble—and I'm glad I did. Kubernetes and the cloud-native ecosystem opened up a whole new layer of infrastructure understanding that complements my developer skills.

If you're a backend engineer thinking about DevOps, cloud-native, or just trying to stay ahead in your career, I highly recommend learning Kubernetes.
