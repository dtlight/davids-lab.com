---
title: "Configuring Alpine Linux on a RPi 4"
date: 2025-12-15T09:09:00+00:00
draft: false
tags: ["talos", "alpine" "raspberry pi 4", "worker node"]
categories: ["worker node"]
hiddenInHomeList: true
---

This blog covers setting up Alpine Linux on a raspberry pi 4 and joining the control plane. Having previously had Talos up and running on a raspberry pi 4, I attempted to re-install and faced numerous boot issues. Additionally, support for Raspberry Pi 5 is minimal and at present cumbersome to get up and running. I have decided not to pour anymore time into running Talos on Raspberry Pi for now. I will however experiment with Talos on x86 architecture in future. Alpine Linux is a lightweight OS like Talos, using minimal memory and effective on Raspberry Pi 4 for idle Kubernetes workloads.

## Prerequisites
Install alpine via raspberry pi imager on your management machine








