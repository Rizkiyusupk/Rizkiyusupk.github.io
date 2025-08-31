---
title: "Deploy Kubernetes 2 Worker - Rolling Update & replicas using manifest "
date: 2025-08-31
categories: [Kubernetes, Deployment]
tags: [k8s, replicas, rolling update, dockerhub, alpine]
header:
  teaser: /assets/images/images.png
---
![logo1](/assets/images/images.png)


## 🌐 Overview

Pada pst kali ini saya akan mendemokan bagaimana cara melakukan **Deployment di Kubernetes v1.28** dengan setup cluster berbasis **Ubuntu 24.04 LTS** ⚡.  
Cluster ini terdiri dari **1 Master Node** dan **2 Worker Node**, sehingga bisa dicoba skenario _real cluster_ sederhana.  

Tujuan dari eksperimen ini adalah:  
- 🔄 **Rolling Update** → mengganti image dari **Alpine v3,19 ➡ v3.22** tanpa downtime  
- 📦 **Replicas** → memastikan aplikasi tetap berjalan meskipun salah satu pod mati  
- ☸️ **Deployment via manifest** → semua konfigurasi dibuat dalam file YAML sehingga bisa dengan mudah dikelola & diulang  

## ⚡ Konsep yang Diterapkan

1. **Deployment 1** → Base image Alpine, dilakukan _rolling update_ dari versi lama ke versi baru dan replicas
2. **Deployment 2** → Menjalankan service dengan beberapa replicas untuk simulasi _load balancing_.  
3. **Deployment 3** → Deploy aplikasi berbasis node.js

Dengan kombinasi ini saya bisa menunjukkan bagaimana Kubernetes:
- Menjaga **aplikasi tetap available** ✅  
- Memungkinkan update versi **tanpa downtime** ✅  
- Memberikan fleksibilitas tinggi dalam **scale up / scale out** ✅  

## ✨ Kenapa Penting?

Dalam dunia **DevOps & Sysadmin**, kita tidak hanya bicara “jalan atau nggak jalannya aplikasi”, tapi bagaimana:  
- 🚦 Aplikasi tetap hidup walaupun ada pod yang mati  
- 🔄 Update bisa dilakukan tanpa ganggu pengguna  
- 📈 Infrastruktur siap di-scale sesuai kebutuhan  

Itulah sebabnya **Deployment + Replicas + Rolling Update** jadi _fundamental skill_ yang wajib dipahami sebelum masuk ke level Kubernetes yang lebih kompleks. 

## Setup dan Conifigurasi

Karena saya akan mendeploy aplikasi menggunakan sebuah manifest maka dari itu bisa dimulai dengan membuat file dengan format **.yml**
[referensi file yaml])(https://www.freecodecamp.org/news/what-is-yaml-the-yml-file-format/)
kamu bisa menulis nama file terserah kamu apapun asal berformat **.yml/.yaml**.Kamu bisa menggunakan text editor apapun **vim** ataupun **nano**

```
vim namafile.yml 
```

atau

```
vim namafile.yaml
```

setelah itu kamu bisa menulis bagian atas dengan isi

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps1
  labels:
    apps1: test
```

kode diatas merupakan bagian awal dari manifest yaitu pembuatan deployment dengan nama apps1 dan berlabel apps1: test,jadi semua pods atau container yang nantinya
di deploy di deployment apps1 bakalan memiliki label atau tag berupa apps1: test,lalu di bagian **apiVersion** itu menandakan semua elemen deployment jadi menggunakan **apps** dan bukan hanya **v1**,lanjut ke bagian deployemnt. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps1
  labels:
    apps1: test
spec:
  replicas: 2
  selector:
    matchLabels:
      apps1: test
  template:
    metadata:
      labels:
        apps1: test
    spec:
      containers:
        - name: container-nginx
          image: alpine:3.19
          command:
            - /bin/sh
            - -c
            - while true; do echo "hidup jokowiii"; sleep 10 ; done;
```

nah tambahan di atas itu adalah bagian dari deployment untuk apps1 seperti **replicas** untuk berapa banyak replicas yang akan digunakan karena saya di atas sudah menyebutkan soal
replicas nah itu dia si **replicas** dalam manifest di tulis seperti itu,lalu bagian selector berfungsi agar si pod masuk ke deployment yang berlabel **apps1: test** karena
memang ditujukan untuk deployment **apps1: test**,trus karena saya juga akan mendemonstrasikan bagaimana cara melakukan rolling update saya menggunakan **alpine** versi 1.19 dan nantinya
akan di update ke versi 1.22.Untuk selanjutnya setelah bagian penulisan untuk deployment apps1,bisa masuk lansgsung ke deployment apps2,untuk rolling up nya bisa nanti setelah semua 
deployment selesai jadi satu manifest memuat 3 deployment,di bagian bawah deployment tekan enter lalu ketik **---** ini sebagai pemisah diantara deployment agar tidak bingung

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps2
  labels:
    apps2: test
spec:
  selector:
    matchLabels:
      apps2: test
  template:
    metadata:
      labels:
        apps2: test
    spec:
      containers:
        - name: container-test
          image: rizki736/test_image
          ports:
            - containerPort: 8081
---
apiVersion: v1
kind: Service
metadata:
  name: apps2-service
spec:
  selector:
    apps2: test
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 80
      nodePort: 30001
  type: NodePort
```

Config di atas itu untuk deployment apps2 dimana saya memakai image custom dari registry di dockerhub yabg sebelumnya sudah saya buat lalu saya push ke regist pribadi saya
dan tentunya jangan lupa dengan service sebagai expose agar nanti bisa di curl lewat cli,di configurasinya saya memakai protocol **tcp** agar bisa saling berkomunikasi antar pod,[selengkapnya mengenai tcp](https://en.wikipedia.org/wiki/Transmission_Control_Protocol),selanjutnya itu saya memakai target port 80 karena base dari image ini itu nginx yang dimana default listennya itu ada di 80, selanjutnya untuk port dari si pods ke node saya buka di 8081 agar nanti bisa berkomunikasi lewat port itu,trus agar bisa di curl lewat master saya pakai 
tipe **node port** dengan port 30001 [referensi node port](https://www.tkng.io/services/nodeport/#:~:text=NodePort%20builds%20on%20top%20of,values%20can%20remain%20the%20same.),kenapa 
saya memakai port 30001 kaya banyak sekali begitu,itu karena alasan range di dalam portnya itu diantara 30000-327627 dan [ini referensinya](https://stackoverflow.com/questions/63698150/in-kubernetes-why-nodeport-has-default-port-range-from-30000-32767),selanjutnya untuk deployment apps3,

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps3
  labels:
    apps3: test
spec:
  selector:
    matchLabels:
      apps3: test
  template:
    metadata:
      labels:
        apps3: test
    spec:
      containers:
        - name: container-node
          image: rizki736/todo
          ports:
            - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: apps3-service
spec:
  selector:
    apps3: test
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 30002
  type: NodePort

```

nah kurang lebih sama dengan deploment apps2 tidak jauh berbeda ada deployment dan service untuk setiap deployment,kenapa di deployment apps1 tidak menggunakan service karena saya sengaja untuk,karena image yang saya gunakan juga sama-sama custom jadi base image ini dari node js makanya saya sengaja target port ke 3000,nah jika sudah exit dari text editornya lalu ketik command ini

```
kubectl apply -f namafile.yml
```

