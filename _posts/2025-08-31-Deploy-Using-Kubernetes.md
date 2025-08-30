---
title: "🚀 Demo Kubernetes Deployment v1.28 - Rolling Update & Replicas"
date: 2025-08-31 02:30:00 +0700
categories:
  - Kubernetes
  - Deployment
tags:
  - kubernetes
  - deployment
  - rolling-update
  - replicas
  - alpine
---

## 🌐 Overview

Pada tulisan kali ini saya akan mendemokan bagaimana cara melakukan **Deployment di Kubernetes v1.28** dengan setup cluster berbasis **Ubuntu 24.04 LTS** ⚡.  
Cluster ini terdiri dari **1 Master Node** dan **2 Worker Node**, sehingga bisa dicoba skenario _real cluster_ sederhana.  

Tujuan dari eksperimen ini adalah:  
- 🔄 **Rolling Update** → mengganti image dari **Alpine v1.19 ➡ v1.22** tanpa downtime  
- 📦 **Replicas** → memastikan aplikasi tetap berjalan meskipun salah satu pod mati  
- ☸️ **Deployment via manifest** → semua konfigurasi dibuat dalam file YAML sehingga bisa dengan mudah dikelola & diulang  

## ⚡ Konsep yang Diterapkan

1. **Deployment 1** → Base image Alpine, dilakukan _rolling update_ dari versi lama ke versi baru.  
2. **Deployment 2** → Menjalankan service dengan beberapa replicas untuk simulasi _load balancing_.  
3. **Deployment 3** → Deploy aplikasi berbasis Nginx dengan akses menggunakan NodePort.  

Dengan kombinasi ini saya bisa menunjukkan bagaimana Kubernetes:
- Menjaga **aplikasi tetap available** ✅  
- Memungkinkan update versi **tanpa downtime** ✅  
- Memberikan fleksibilitas tinggi dalam **scale up / scale out** ✅  

## ✨ Kenapa Penting?

Dalam dunia **DevOps & Sysadmin**, kita tidak hanya bicara “jalan atau nggak jalannya aplikasi”, tapi bagaimana:  
- 🚦 Aplikasi tetap hidup walaupun ada pod yang mati  
- 🔄 Update bisa dilakukan tanpa ganggu pengguna  
- 📈 Infrastruktur siap di-scale sesuai kebutuhan  

Itulah sebabnya **Deployment + Replicas + Rolling Update** jadi _fundamental skill_ yang wajib dipahami sebelum masuk ke level Kubernetes yang lebih kompleks. 🔥
