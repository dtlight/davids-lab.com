---
title: "Kubernetes Releases v1.35"
date: 2025-12-30T09:09:00+00:00
draft: false
tags: ["timbernetes", "1.35", "release"]
categories: ["releases"]
hiddenInHomeList: true
---

![Timbernetes](https://kubernetes.io/blog/2025/12/17/kubernetes-v1-35-release/k8s-v1.35.png)


# Kubernetes Releases version 1.35 

On 17 December 2025 Kubernetes released version 1.35 code named “Timbernetes”, which is presented as a turning-point release that modernizes cluster operations, tightens security, and brings long-awaited vertical scaling and AI-focused scheduling into the core platform. Timbernetes has a theme of deepening the platform’s security posture while simplifying the operational model for platform teams.

## Release theme and scope

Timbernetes leans into a “world tree” metaphor to emphasize maturity, deep roots in stability/security, and new “branches” for AI and edge workloads. The episode frames the release as one that will change how production clusters are run, highlighting 60+ enhancements and several high‑impact deprecations and removals.

## Three breaking changes
There are at least three breaking changes that cluster operators must plan for before upgrading.
​
* **cgroup v1 removed:** Kubelet will not start on nodes still using cgroup v1, forcing a move to cgroup v2 for all supported environments. This affects older OS images and any custom AMIs that never migrated their control group configuration.

* **containerd 1.x end of life:** Kubernetes 1.35 is the last release that will run with containerd 1.7, so teams must plan upgrades to containerd 2.x or risk being unable to move to future Kubernetes versions. This is particularly risky for air‑gapped or slow‑moving enterprises that pin old runtime versions.
​
* **IPVS mode deprecated:** IPVS service proxying is deprecated in favor of iptables/nftables‑based implementations. Operators relying on IPVS for performance or sticky sessions are urged to try alternatives and adjust cluster networking configurations early.

## In‑Place Pod Resize GA (headline feature)

In‑Place Pod Resize, which finally reaches GA (General Availability, ie 'stable') after roughly six years from the initial KEP‑1287. The core change is that `spec.containers[].resources` for CPU and memory is now mutable, backed by a new resize sub-resource so the kubelet can adjust cgroups without restarting pods.
​
​Key behaviors and use cases include:
​
* **Zero‑downtime vertical scaling**: CPU and memory can be increased or decreased on running pods without recreating them, avoiding connection drops, cache loss, or long warm‑up times.

* **Safe memory downsizing:** Memory limit decreases are now supported with kubelet best‑effort checks, enabling right‑sizing of over-provisioned workloads instead of waiting for a reschedule.

* **New patterns:**

    - Autoscalers or operators that continuously right‑size pods based on SLOs or internal metrics.
    - Batch jobs that temporarily request more resources for intensive phases, then shrink back.
    - Platforms that offer self‑service “boost” or “burst” controls to application teams.
​
​This is foundational for cost optimization in multi‑tenant clusters and an enabler for dynamic resource management in AI/ML and data workloads.

## Pod Certificates Beta and security hardening
Pod Certificates, a new beta feature (KEP‑4317) brings native, kubelet‑managed mTLS to pod‑to‑pod communication. Instead of relying on cert‑manager, SPIRE, or other sidecar‑based approaches, the kubelet now provisions and rotates workload certificates tied directly to pod identity.
​
​Security outcomes include:

* Built‑in zero‑trust foundations: Every pod can get an X.509 identity, enabling mutual TLS inside the cluster without a separate PKI stack.

* Stronger node impersonation defences: Improved certificate handling and node validation reduce the risk of rogue machines joining a cluster and exfiltrating secrets or traffic from legitimate workloads.
​
## Gang Scheduling Alpha and AI workloads

Kubernetes 1.35 adds alpha support for gang scheduling via a new workload‑aware scheduling API, aimed squarely at distributed AI/ML training and other tightly coupled batch workloads. Gang scheduling can be described as “all or nothing”: either the full group of interdependent pods is scheduled together, or none are. 

This:

* Eliminates partial‑start failures where only some workers come up, causing idle GPU nodes and failed training runs.

* Allows expressing job‑level requirements (e.g., “8 GPUs across 2 nodes”) directly to the scheduler, without custom controllers coordinating reservations.
​
* Opens the door for native AI/ML workload platforms to rely less on external schedulers like Volcano or bespoke operator logic.
​
## Other alpha features and scheduling enhancements
Beyond the headline items, there are several alpha and beta features that strengthen workload placement and hardware awareness.
​
* Node Declared Features: Nodes can formally advertise capabilities (e.g., NUMA layout, accelerators, storage tiers), giving the scheduler richer data for placement decisions.
​
* Partitionable Devices: Better modelling of devices that can be sliced or shared (for example, GPUs, DPUs, or accelerators), improving density and resource utilisation.
​
* Extended Toleration Operators: New taint/toleration operators beyond just Equal and Exists, enabling SLO‑aware or “threshold” style placement such as targeting nodes above a certain cost, reliability, or performance score.
​
​These are all presented as steps toward “workload‑aware scheduling”, where the platform understands more about both hardware and SLOs, not just simple resource requests. You can read the full detail here https://kubernetes.io/blog/2025/12/17/kubernetes-v1-35-release/.
