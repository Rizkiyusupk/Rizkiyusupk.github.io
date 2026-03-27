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
- **Rhel (plow)** : 9.6
- **Kubernetes** : v1.30
- **Container Runtime Interface** : crio 1.30.10
- **CNI Plugin** : Flannel
- **Vmware Workstation** : 25h2
- **Docker** : 28.2.2

  
| Node        | CPU     | RAM  | Storage | Network                               |
|-------------|---------|------|---------|---------------------------------------|
| **Master**  | 5 cores | 8GB  | 25GB    | 2 Adapters (NAT + Host-Only-Adapter)  |
| **Worker 1**| 5 cores | 6GB  | 25GB    | 2 Adapters (NAT + Host-Only-Adapter)  |
| **Worker 2**| 5 cores | 6GB  | 25GB    | 2 Adapters (NAT + Host-Only-Adapter)  |



### Setup Rhel
pertama masuk ke Vmware terlebih dahulu lalu klik create virtual machine 

![asdin](/assets/images/redhatt/Screenshot 2026-03-22 212852.png)

kurang lebih seperti diatas tampilannya,lalu akan muncul tampilan pop up untuk opsi custom atau typical,

![asidj](/assets/images/redhatt/Screenshot 2026-03-23 004932.png)

pilih custom,karena kita akan membuat sebuah virtual machine yang mumpuni,jika sudah next
kamu akan disuruh untuk memilih hardware compatibility 

![asd](/assets/images/redhatt/Screenshot 2026-03-23 004936.png)

biarkan saja itu default mengikuti versi dari si vmware nya 
selanjutnya kamu akan disuruh untuk memasukan disc atau image yang dibutuhkan checklist bagian installer disc image file 

![ausdb](/assets/images/redhatt/Screenshot 2026-03-23 004940.png)

lalu pilih image yang sebelumnya sudah di download,tenang saja dengan warning yang ada di bawah nanti di tahap selannjutnya kita masukan versi
dari rhel nya,next adalah untuk memilih versi dari rhel image yang sebelumnya sudah di masukan 

![aishdasnd](/assets/images/redhatt/Screenshot 2026-03-23 004947.png)

pilih **Red Hat Enterprise Linux 9 64-bit**  itulah versi yang akan digunakan dalam tutorial ini,
selanjutnya adalah memilih seberapa banyak prosesor yang akan digunakan, 

![asuda](/assets/images/redhatt/Screenshot 2026-03-23 004951.png)

untuk prosesor sendiri gunakan 5 prosesor dengan cores per prosesornya itu 1
masuk ke tahap selanjutnya yaitu memlilih seberapa besar memory yang akan digunakan dalam cluster 

![asihd](/assets/images/redhatt/Screenshot 2026-03-23 004954.png)

untuk node master akan menggunakan memori 8gb dan untuk node worker

![asidh](/assets/images/redhatt/Screenshot 2026-03-27 210849.png)

gunakan 6 gb untuk semua node worker,selanjutnya ialah tipe jaringan ,
jaringan yang akan digunakan itu ada dua nantinya 

```
NAT & HOST ONLY ADAPTER
```

untuk yang pertama terlebih dahulu yaitu nat ,nat berfungsi agar virtual mesin nantinya bisa mengakses internet 
dan untuk host only adapter itu agar virtual mesin bisa diakses dan berkomunikasi dengan host device yaitu laptop atau komputer

![uasdb](/assets/images/redhatt/Screenshot 2026-03-23 005006.png)

nah diatas merupakan contoh pop up windows nya nanti,kenapa hanya satu nat saja? karena ini hanya sementara nanti 
akan ditambahkan adapter baru ketika step installasi ini selesai 
selanjutnya ialah bagian setup i/o 

![asidha](/assets/images/redhatt/Screenshot 2026-03-23 005010.png)

untuk bagian ini biarkan default saja,selanjutnya ialah tipe disk 

![aishd](/assets/images/redhatt/Screenshot 2026-03-23 005015.png)

sama bagian ini juga biarkan default saja,lalu selanjutnya 

![aishd](/assets/images/redhatt/Screenshot 2026-03-23 005026.png)

untuk membuat disk baru pilih saja create new disk setelah itu kamu akan diarahkan ke pop up selanjutnya 

![asjd](/assets/images/redhatt/Screenshot 2026-03-23 005036.png)

untuk ukuran dari disk di semua node itu sama 25gb jadi sama rata, dan pastika disk di satu folder jangan split

![iahsd](/assets/images/redhatt/Screenshot 2026-03-23 005041.png)

arahkan ke nama dari disk biarkan default saja nanti juga bisa di rename,

![ajsbd](/assets/images/redhatt/Screenshot 2026-03-23 005045.png)

dan tahap terakhir yaitu pembuatan virtual mesin klik finish dan vmware otomatis akan membuat virtual mesin nya,

### Install Rhel

Masuk ke tahap selanjutnya yaitu bagian installasi rhel , di step ini khusus membahas soal step by step installasi rhel


