---
title: "Resource Planning"
date: 2025-12-13T09:09:00+00:00
draft: false
tags: ["planning"]
categories: ["design"]
hiddenInHomeList: true
---


## Resource Planning
My four node cluster currently consists of:
1. Two control plane nodes (cpl1 and cpl2 16gb ram): both running Raspberry Pi OS (Debian 13) in headless mode so are lightweight and Pi optimised. Etcd requires the most RAM (3GB for small clusters like mine with <100 pods), while other components should typically stay under 1GB each (with Prometheus at roughly 3gb as an exception) with my multi-node control plane set up which distributes load, so 16GB handles 20-50 pods across nodes without OOM issues.
2. Two worker nodes (wn1, wn2), one runs Alpine and the other Debian. Alpine Linux is a highly secure yet minimal operating system. 

I have mixed 8GB and 40GB worker nodes which work fine for my Kubernetes homelab, as schedulers automatically balance pods based on allocatable resources (node taints/labels also help). 8GB should handle 20-40 pods (e.g., Prometheus targets, Home Assistant), while 40GB can take heavier workloads like databases or future AI experiments without OOM issues.​

Having two control plane nodes promotes High Availability (H.A) for the things I will run on my cluster, which would be frustrating to have offline:
* [HomeAssistant](https://hub.docker.com/r/homeassistant/home-assistant) 
* [AdGuard ](https://hub.docker.com/r/adguard/adguardhome) (home dns server)

To achieve H.A in a real world production environment using a hybrid cloud/on premise or cloud/cloud (eg aws/gcp) setup would be obviously more sensible than a cluster of raspberry pis. Also, my control plane nodes share the same ip via kube-virtual-ip which is owned by one of my control plane nodes. In a production environment this would be its own dedicated machine or better still a load balancer instead of kube-vip.

I will create two namespaced environments, dev and prod. No pod in either environment runs on a control plane node.

| Node / Type | RAM Allocatable | Pod Capacity | Example Workloads |
|-----------|-----------------|--------------|------------------|
| **16GB** / CP1 | ~14GB | 50-80 pods | Etcd, Prometheus |
| **16GB** / CP2 | ~14GB | 50-80 pods | Same as CP1 |
| **8GB** / Worker 1 (arm) | 6GB | 20-40 pods | Pi-hole, Home Assistant monitoring agents, lightweight apps|
| **40GB** / Worker 2 (x86)| 36GB | 100-200 pods  | StatefulSets, Postgresql, heavy services|
