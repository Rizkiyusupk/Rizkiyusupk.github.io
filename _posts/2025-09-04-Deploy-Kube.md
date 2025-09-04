---
title: "Deploy Kubernetes 2 Worker - Performing configMap configuration & init containers with rolling update "
date: 2025-09-04
categories: [Kubernetes, Deployment]
tags: [k8s, initContainers, rolling update, configMap, alpine]
header:
  teaser: /assets/images/images.png
---
![logo](/assets/images/images.png)

## 🔄 Rolling Update dengan ConfigMap di Kubernetes

Hari ini saya eksperimen bagaimana **ConfigMap** di Kubernetes bisa dipakai untuk mengatur konfigurasi aplikasi ✨.  
Hasilnya? Saya bisa ganti isi halaman **nginx** hanya dengan update ConfigMap — tanpa rebuild image sama sekali! 🚀  

### 📝 Langkah yang saya lakukan:
1. Buat **Deployment** dengan container nginx 📦  
2. Mount **ConfigMap** ke dalam Pod 🗂️  
3. Akses via **NodePort** untuk lihat hasil awal 🌐  
4. Update ConfigMap → hapus Pod lama → Pod baru otomatis pakai config terbaru 🔄  

## 🎯 Tujuan dari Latihan Ini

Kenapa sih perlu belajar rolling update dengan ConfigMap? 🤔  

- 🔧 **Dynamic Config** → Aplikasi bisa ganti konfigurasi tanpa rebuild image.  
- ⏱️ **Hemat waktu** → Nggak perlu bikin image baru tiap kali ada perubahan kecil.  
- 🌀 **Rolling Update mulus** → Config baru langsung dipakai Pod baru tanpa downtime yang lama.  
- 📦 **Best Practice Kubernetes** → Memisahkan *config* dari *code* adalah prinsip dasar agar aplikasi lebih fleksibel & mudah di-manage.  

Dengan kata lain, latihan ini ngajarin mindset bahwa:  
👉 **image = aplikasi**,  
👉 **ConfigMap = konfigurasi**,  
dan keduanya sebaiknya dipisah untuk skalabilitas jangka panjang ⚡.

### 🎉 Hasil
Begitu ConfigMap diganti, halaman nginx langsung berubah warnanya sesuai isi ConfigMap baru 🌈.  
Rolling update berjalan mulus tanpa downtime yang berarti 🙌.  

💡 **Insight:**  
Dengan cara ini, kita bisa ubah konfigurasi aplikasi kapan saja tanpa harus rebuild image atau redeploy penuh. Super praktis buat aplikasi yang konfigurasi-nya sering berubah ⚡.
