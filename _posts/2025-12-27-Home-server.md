---
title: "Building a Lightweight Kubernetes Home Server with k3s (CLI-Only Setup) Using Bare Metal"
date: 2025-12-27
tags: [kubernetes, k3s, home-server, linux, cli, devops, containerd, nginx,home-server]
header:
  teaser: /assets/images/home-server/18555159.png
categories: [DevOps, Kubernetes,linux,Linux,Server]
---

![anjay](/assets/images/home-server/18555159.png)

## Overview

Pada dokumentasi ini, saya menjelaskan proses mengubah sebuah laptop lama yang sudah jarang digunakan menjadi single-node Kubernetes home server menggunakan k3s, tanpa bergantung pada virtual machine maupun antarmuka grafis (GUI).

Alih-alih menjalankan Kubernetes di dalam beberapa VM—yang menambah beban resource pada perangkat,dengan spesifikasi terbatas setup ini dijalankan langsung di bare metal dengan lingkungan Ubuntu CLI-only. Pendekatan ini bertujuan untuk mendapatkan sistem yang ringan, stabil, dan tetap mendekati kondisi production untuk skala rumahan.

## Software Version
- **OS: Ubuntu 25.04**
- **Kubectl: v1.35.0**
- **Docker: 28.2.2**


## Setup
Dikarenakan ini menggunakan bare metal dan bukan vm disini tidak ada setup vm seperti biasanya,jadi saya menggunakan laptop lama saya yang 
sudah jarang di gunakan sebagai single node dan kebetulan os dari laptop lama saya itu ubuntu jadinya saya bisa memanfaatkan laptop usang saya 
sebagai bahan belajar,dan kenapa saya memilih untuk k3s bukan k8s karena konsep saya itu single node jadi saya memakai k3s kenapa bukan k3d? 
nanti di postingan selanjutnya saya akan menggunakan k3d,jadii langsung ke tutorialnya

Pertama-tama karena laptop lama saya itu os ubuntu tapi GUI dan bukan CLI saya akan merubahnya terlebih dahulu 
gunakan command 

```
sudo systemctl set-default multi-user.target
sudo reboot
```

setelah itu os akan reboot dan akan masuk ke mode CLI 

dari yang tadinya GUI 

![logoo](/assets/images/home-server/WhatsApp Image 2025-12-28 at 18.56.59.jpeg)

ke CLI

![looo](/assets/images/home-server/Screenshot 2025-12-28 191127.png)

jika ingin kembali ke tampilan GUI dari CLI ketik command

```
sudo systemctl set-default graphical.target
sudo reboot
```

kenapa saya menggunakan CLi daripada GUI karena masalah beban pada laptop karena laptop ini laptop usang yang sudah lama jadinya saya menggunakan
CLI untuk mengurangi beban dan ini hanya untuk pembelajaran semata untuk saya agar saya paham juga menggunakan bare metal tidak selalu vm,
jika sudah langsung masuk ke terminal windowsnya lalu paste command ini

```
curl -sfL https://get.k3s.io | bash 
```

saya menggunakan install script jadinya buka yang binary biar lebih cepat dan praktis,
jika sudah maka akan ada output yang mirip seperti ini

![logoo](/assets/images/home-server/Screenshot 2025-12-28 192010.png)

output diatas adalah output dari laptop saya yang sudah terinstall k3s jadinya banyak kata **SKIPPING** jika pertama kali
install maka akan ada output **INFO**,jika sudah test cek nodes 

```
kubectl get nodes
```

jika sudah mengetikan command seperti diatas maka output akan seperti ini

![logooo](/assets/images/home-server/Screenshot 2025-12-28 232047.png)

kenapa rolesnya hanya ada  control-plane,master   dan 1 node ya karena ini single node jadi hanya ada 1,misalkan 
ada bare metal tambahan tinggal join menggunakan command 

```
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```

ganti url dengan alamat apiserver master atau ip dari node master lalu untuk node tokennya ambil dari master menggunakan command

```
sudo cat /var/lib/rancher/k3s/server/node-token

```

SELESAI SUDAH DEMO INSTALLASI SAYA MENGGUNAKAN BARE METAL INI MERUPAKAN LANGKAH PERTAMA SAYA
MENGGUNAKAN BARE METAL KARENA SAYA SERING MENGGUNAKAN VM TERIMAKASIH SUDAH MENYIMAK DADAHH!!!
