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
4. Update ConfigMap 🔄  

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


## Setup Configuration

Baik untuk selanjutnya bisa langsung masuk ke langkah selanjutnya yaitu penulisa manifest [selengkapnya untuk manifest](https://spacelift.io/blog/kubernetes-manifest-file)
sebelum dimulai buat configmapnya terlebih dahulu,

```
kubectl create configmap game --from-literal=game=minikrep
```

dikarenakan saya akan membuat dua deployment saya akan membuat dua configmap

```
kubectl create configmap presiden --from-literal=presiden=jokowi
```

kamu bisa pakai vim atau nano untuk text editornya, dan format dari filenya itu yml ya aku sudah pernah ngasih tau di postingan sebelumya

```
vim nama_file.yml
```

atau jika pakai nano 

```
nano nama_file.yml
```

jika sudah langsung tulis bagian head dari si deployment dan deployment ini akan dibagi menjadi dua bagian deployment apps1 dan apps2

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps1
  labels:
    apps1: test
```

nah config diatas itu untuk membuat sebuah deployment dengan nama apps1 dan label **apps1: test** untuk semua pods dan container yang masuk ke deployment apps1
untuk selanjutnya bagian dari si deploymentnya

```
spec:
  selector:
    matchLabels:
      apps1: test
  template:
    metadata:
      labels:
        apps1: test
    spec:
      volumes:
        - name: shared-data
          emptyDir: {}
        - name: config-volume
          configMap:
            name: game
            defaultMode: 0777
      containers:
        - name: container-nginx
          image: nginx:latest
          volumeMounts:
            - name: shared-data
              mountPath: /usr/share/nginx/html/
          ports:
            - containerPort: 8080
        - name: container-alpine
          image: alpine:latest
          volumeMounts:
            - name: config-volume
              mountPath: /etc/config
            - name: shared-data
              mountPath: /pod-data/
          env:
            - name: GAME
              valueFrom:
                configMapKeyRef:
                  name: game
                  key: game
          command:
            - /bin/sh
            - -c
            - while true; do echo "saya cinta $( cat /etc/config/game )" > /pod-data/index.html; sleep 10 ; done;
```

nah untuk di deploymentnya sendiri itu saya menggunakan 2 container 1 pods dan dua duanya itu sidecar container,jadi container pertama akan berfungsi sebagai webserver dan yang satunya lagi berfungsi sebagai output untuk nantinya digunakan di webserver,nah container pertama menggunakan nginx sebagai image dan yang kedua menggunakan alpine sebagai image, 
[nginx](https://nginx.org/) & [alpine](https://www.alpinelinux.org/),karena nantinya si container kedua akan dibutuhkan untuk menulis file html yang nantinya dipakai oleh si webserver,si file akan berisikan output dari configmap yang sebelumnya dibuat lalu, akan ditulis ke dalam sebuah direktori bernama **/pod-data** yang sudah di mount ke sebuah volume kosong bernama
**shared-data** nah di shared data inilah dua container saling bertukar informasi dan di container nginx si default untuk file statis nginx ditimpa ke **shared-data** yang nantinya 
akan menunjukan output dari si file html yang di buat container alpine,nah di situ letak perubahan untuk container nginx tanpa remove atau build ulang,nahh si output itu bisa diedit lewat 
**configmap edit** jadi tinggal edit configmapnya ga usah buat ulang dari awal dan bisa masukin beberapa value buat si configmapnya jadi output dari si webserver bisa di edit tanpa build dari awal, dan karena ini itu sidecar container jadi si container alpine akan terus menerus berputar-putar atau melakukan **while true**
