---
title: "Blogs"
layout: "posts" 
draft: false
---

Below you can view a complete list of my uploaded blogs. I have also a little table showing some detail about what my cluster consists of. 

For more information see resource planning.

| Node / Type | RAM Allocatable | Pod Capacity | Example Workloads |
|-----------|-----------------|--------------|------------------|
| **16GB** / CP1 | ~14GB | 50-80 pods | Etcd, Prometheus |
| **16GB** / CP2 | ~14GB | 50-80 pods | Same as CP1 |
| **16GB** / Worker 1 | ~14GB | 50-80 pods | StatefulSets, Postgresql, heavy services|
| **8GB** / Worker 2 | ~6GB | 20-40 pods | Pi-hole, Home Assistant monitoring agents, lightweight apps|
