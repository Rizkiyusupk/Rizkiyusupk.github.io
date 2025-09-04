---
title: "Deploy Kubernetes 2 Worker - Performing configMap configuration & init containers, sidecar containers with rolling update "
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
**configmap edit** jadi tinggal edit configmapnya ga usah buat ulang dari awal dan bisa masukin beberapa value buat si configmapnya jadi output dari si webserver bisa di edit tanpa build dari awal, dan karena ini itu sidecar container jadi si container alpine akan terus menerus berputar-putar atau melakukan **while true**, nah untuk selanjutnya gunakan service di manifest
supaya si nodenya bisa di akses 

```
---
apiVersion: v1
kind: Service
metadata:
  name: apps1-service
spec:
  selector:
    apps1: test
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
      nodePort: 30001
  type: NodePort
```

disini saya memakai port 30001 soalnya itu masuk dalam range port untuk pods kube,untuk selanjutnya saya akan menggunakan initcontainers deployment jika apps1 deployment itu sidecar container nah yang deployment apps2 itu initcontainers, [initcontainers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) & 
[sidecar containers](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/),untuk selanjutnya langsung masuk ke bagian head dari deployment apps2

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps2
  labels:
    apps2: test
```

nah kurang lebih sama sih dengan cuman beda di name deploymentnya sama labelsnya, untuk selanjutnya masuk ke bagian deploymentnya

```
spec:
  selector:
    matchLabels:
      apps2: test
  template:
    metadata:
      labels:
        apps2: test
    spec:
      volumes:
        - name: shared-data
          emptyDir: {}
        - name: config-volume
          configMap:
            name: presiden
            defaultMode: 0777
      containers:
        - name: container-nginx
          image: nginx:latest
          volumeMounts:
            - name: shared-data
              mountPath: /usr/share/nginx/html/
          ports:
            - containerPort: 8081
      initContainers:
        - name: container-alpine
          image: alpine:3
          volumeMounts:
            - name: shared-data
              mountPath: /pod-data/
            - name: config-volume
              mountPath: /etc/config/
          env:
            - name: GAME
              valueFrom:
                configMapKeyRef:
                  name: presiden
                  key: presiden
          command:
            - /bin/sh
            - -c
            - echo " $(cat /etc/config/presiden) anjing" > /pod-data/index.html
```

sama tidak ada bedanya dengan depolyment apps1 cuman disini itu ada bagian init containers,nah apasih bedanya init containers dengan sidecar containers,bedanya itu
jika si init containers sudah di eksekusi si manifest langsung masuk ke container utama yaitu si nginx container,nah si inti container ini sifatnya sementara dan akan terhapus otomatis
jika sudah menjalankan tasknya misa di dalam contoh ada command echo nah jika si command echo sudah dijalankan tanpa ada eror si container akan terhapus dan masuk ke container utama yaitu
si nginx,ibaratkan si init containers ini permulaan agar si container utama jalan soalnya jika si init eror manifest tidak akan mengeksekusi container utama,tidak seperti sidecar containers
yang permanent menjalankan contanier bersamaan dengan container utama,maka dari itulah alasan saya menggunakan **while true** di deployment apps1 karena itu adalah sidecar container,jadinya tidak akan hilang sementara saya cuman memakai echo command biasa di deployment apps2 karena memang jenisnya initcontainers jadi jika ada looping seperti **while true**
akan terjadi eror yang menyebabkan si deployment stuck di init containers,jika sudah tambahkan service seperti biasa

```
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
      nodePort: 30002
  type: NodePort
```

jika sudah selesai dengan manifestnya bisa langsung apply si filenya menggunakan command **kubectl apply -f nama_file**

```
kubectl apply -f nama_file.yml
```

jika sudah coba cek outputnya terlebih dahulu

![logo1](/assets/images/Kube/memek8.png)

jika sudah mendapat informasi mengenai containernya tinggal informasi mengenai ip dari si node supaya nanti bisa melakukan curl ke si container

![logo2](/assets/images/Kube/memek7.png)

nahh jika sudah mendapatkan informasi ip dari si nodenya tinggal curl deh pertama-tama coba curl untuk apps1 atau sidecar container,

![logo3](/assets/images/Kube/memek5.png)

jika suda curl kamu bisa langsung melihat outputnya yaitu value dari configmap bernama game dengan value mobellegend
jika sudah ada coba ganti outputnya menggunakan command **kubectl edit configmap nama_configmap**

```
kubectl edit configmap nama_configmap
```

jika sudah outputnya akan seperti ini

![logo4](/assets/images/Kube/memek4.png)

jika sudah ganti nilai dari si variable game menjadi apapun terserah disini saya menggunakan minikrep

![logo5](/assets/images/Kube/memek3.png)

nah kurang lebih seperti ini,jika sudah exit lalu coba curl dan lihat outputnya

[logo6](/assets/images/Kube/memek2.png)

dan begitulah kurang lebih demonstrasi mengenai update isi container untuk sidecar container,
dan untuk init container cuman curl biasa untuk mengecek apakah initnya sudah jalan 

![logo7](/assets/images/Kube/memek1.png)

jika ada output dari curl berarti initnya sudah jalan 
🚀🚀🚀🚀  
