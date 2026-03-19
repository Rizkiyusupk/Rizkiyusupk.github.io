---
title: "Build Kubernetes Cluster in Rhel"
date: 2026-03-19
tags: [kubernetes, linux, Ci/Cd, devops, containerd,ci,cd,ci/cd,system,infrastructure,Redhat,rhel]
header:
  teaser: /assets/images/redhat1/Untitled design (4).jpg
categories: [DevOps, Kubernetes,linux,Linux,Server,Deploy,Deployment]
---

![asd](/assets/images/redhat1/Untitled design (4).jpg)

### Overview 

Postingan kali ini saya akan membahas bagaimana cara membuat kubernetes cluster di sistem operasi rhel (Red Hat enterprise Linux) ,
nah postingan sebelum-sebelumnya itu kan menggunakan Ubuntu, untuk sekarang saya akan menggunakan rhel distro yang berbeda,dan kedepanya juga 
akan ada distro distro yang lain,tapi saya akan secara khusus menbahas distro ini , ada banyak sekali kemiripan waktu install dengan distro ubuntu.
Langsung saja masuk ke stepnya

### Tools
- **Rhel (0otpa)** : 8
- **Kubernetes** : v1.28.15
- **Container Runtime Interface** : containerd v2 2.1.3
- **CNI Plugin** : Flannel
- **VirtualBox** : 7.0.10
- **Docker** : 28.2.2

### Setup 
pertama 
