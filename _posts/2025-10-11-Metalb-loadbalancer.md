---
title: "Kubernetes Load Balancer Setup with MetalLB on Multi-Node Cluster (3 VMs) "
date: 2025-09-04
categories: [Kubernetes, Metalb,LoadBalancer]
tags: [k8s, initContainers, rolling update, configMap, alpine]
header:
  teaser: /assets/images/images.png
---
![logo](/assets/images/images.png)


# ☸️ Instalasi MetalLB sebagai Load Balancer pada Cluster Kubernetes (1 Master, 2 Worker)

Repositori ini berisi dokumentasi proses saya dalam menginstal dan mengonfigurasi **MetalLB** sebagai **Load Balancer** untuk cluster Kubernetes berbasis **VM lokal**.  
Karena cluster ini tidak dijalankan di cloud provider (seperti AWS, GCP, atau Azure), Kubernetes secara default tidak dapat memberikan IP eksternal pada Service tipe **LoadBalancer**.  
Untuk mengatasinya, digunakan **MetalLB** agar service dapat memiliki **alamat IP eksternal (external IP)** dan diakses dari jaringan lokal.

---

## 🎯 Tujuan Proyek
Proyek ini dibuat dengan tujuan untuk:
- ⚙️ Mengaktifkan fitur **LoadBalancer Service** pada cluster vm (Virtual Machine)
- 🌐 Memungkinkan akses aplikasi dari luar cluster menggunakan **IP statis**  
- 📘 Mempelajari cara kerja **networking Kubernetes** secara lebih mendalam  

---

## 💡 Mengapa Menggunakan MetalLB?
- ☸️ Kubernetes tidak menyediakan Load Balancer bawaan di lingkungan non-cloud  
- 🌍 **MetalLB** memungkinkan Service tipe LoadBalancer mendapatkan IP eksternal  
- 🔧 Mendukung dua mode operasi: **Layer 2 (L2)** dan **BGP**  
- 💡 Mudah dipasang, cocok untuk **VM lokal**, **lab pengujian**, maupun **on-premise cluster**

Tanpa MetalLB, Service tipe LoadBalancer akan selalu berstatus `<pending>` karena tidak ada komponen yang menyediakan IP eksternal.

---

## 🧱 Detail Cluster

| 🖥️ Node | ⚙️ Peran | 🐧 Sistem Operasi | 💾 Kapasitas Disk | ☸️ Versi Kubernetes |
|:--------|:----------|:-----------------|:------------------|:--------------------|
| **Master** | Control Plane | Ubuntu 24.04 LTS | 30 GB | v1.28 |
| **Worker 1** | Node Worker | Ubuntu 24.04 LTS | 30 GB | v1.28 |
| **Worker 2** | Node Worker | Ubuntu 24.04 LTS | 30 GB | v1.28 |



- 🧱 **Total Nodes:** 3  
- 🐧 **Operating System:** Ubuntu 24.04 LTS  
- ⚙️ **Container Runtime:** containerd  
- 📦 **Deployed Using:** kubeadm

---
