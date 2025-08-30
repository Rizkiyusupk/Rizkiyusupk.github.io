---
title: "Deploy Pods Kubernetes dengan Replicas & Rolling Update"
date: 2025-08-31
categories: [Kubernetes, Deployment]
tags: [k8s,Kubernetes,Deployment]
tags: [k8s, replicas, rolling-update, alpine, dockerhub]
header:
  teaser: /assets/images/images.png

---
![kubelogo](/assets/images/images.png)


## 🚀 Overview

Pada percobaan ini, aku melakukan **deployment Kubernetes** menggunakan **Ubuntu 24.04 LTS** dengan **Kubernetes v1.28**.  
Cluster terdiri dari **1 master node** dan **2 worker node** yang sudah saya tulis dokumentasi pembuatan clusternya [disini]()

## 🔧 Konsep yang Diterapkan
- Menggunakan **3 Deployment** dengan **replicas** untuk memastikan aplikasi tetap tersedia ⚡  
- Melakukan **Rolling Update** dari **Alpine v1.19 ➡ Alpine v1.22** tanpa downtime 🔄  
- Menerapkan **custom image** yang sudah dipush ke **Docker Hub** 🐳  

## 📌 Hasil
Dengan setup ini, aku bisa menguji secara langsung:
- Proses **scale out** (menambah replicas)  
- Proses **rolling update** yang berjalan mulus  
- Deployment management yang lebih fleksibel ☸️  

Singkatnya, ini adalah langkah awal untuk memahami bagaimana **Kubernetes Deployment** bisa menjaga aplikasi tetap stabil sekaligus memudahkan update di lingkungan cluster. ✨

