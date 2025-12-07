---
title: "Setup Kubernetes Cluster 1 Master + 2 Worker dengan VirtualBox"
date: 2025-08-16
tags: [kubernetes, vm, virtualbox, containerd, flannel, cluster, devops]
header:
  teaser: /assets/images/ci-cd/Untitled design (2).png
categories: [DevOps, Kubernetes]
---

![Kubernetes Cluster Architecture](/assets/images/ci-cd/Untitled design (2).png)

## 🎯 **Overview**
Membangun Kubernetes cluster production-like dengan 1 Master dan 2 Worker Node menggunakan VirtualBox. Setup ini cocok untuk development, testing, dan pembelajaran Kubernetes.

## 📋 **Spesifikasi Hardware & Software**

### **Virtual Machine Requirements:**
| Node | CPU | RAM | Storage | Network |
|------|-----|------|---------|---------|
| **Master** | 2 cores | 2GB | 20GB | 2 Adapters (NAT + Host-only) |
| **Worker 1** | 2 cores | 2GB | 20GB | 2 Adapters (NAT + Host-only) |
| **Worker 2** | 2 cores | 2GB | 20GB | 2 Adapters (NAT + Host-only) |

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
