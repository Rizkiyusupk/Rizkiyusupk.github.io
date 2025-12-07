---
title: "Build Integrated Deployment System Full Setup"
date: 2025-08-16
tags: [kubernetes, vm, virtualbox, containerd, flannel, cluster, devops,system]
header:
  teaser: /assets/images/ci-cd/Untitled design (2).png
categories: [DevOps, Kubernetes]
---

![Kubernetes Cluster Architecture](/assets/images/ci-cd/Untitled design (2).png)

##  **Overview**
Saya akan membangun Kubernetes cluster production-like dengan 1 Master dan 2 Worker Node menggunakan VirtualBox. Setup ini cocok untuk development, testing, dan pembelajaran Kubernetes.Semua ini bertujuan untuk pembelajaran dan pemahaman semata agar saya dan yang melihat ini bisa paham dalam Ci/Cd,

##  **Spesifikasi Hardware & Software**

### **Virtual Machine Requirements:**

| Node        | CPU     | RAM  | Storage | Network                     |
|-------------|---------|------|---------|------------------------------|
| **Master**  | 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Host-only) |
| **Worker 1**| 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Host-only) |
| **Worker 2**| 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Host-only) |
| **Jenkins** | 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Host-only) |

### **Software Versions:**
- **OS**: Ubuntu Server 24.04 LTS
- **Kubernetes**: v1.28.15
- **Container Runtime**: containerd v1.27.7
- **CNI Plugin**: Flannel
- **VirtualBox**: 7.0.10

## 🚀 **Step 1: VirtualBox Setup**

### **1.1 Download Ubuntu Server ISO**
```bash
# Download Ubuntu 24.04 LTS
wget https://releases.ubuntu.com/24.04/ubuntu-24.04-live-server-amd64.iso
