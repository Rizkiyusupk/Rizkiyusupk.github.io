---
title: "Debugging error build kubernetes on Rhel "
date: 2026-03-30
tags: [kubernetes, linux, Ci/Cd, devops, containerd,ci,cd,ci/cd,system,infrastructure,Redhat,rhel]
header:
  teaser: /assets/images/redhat1/Untitled design (4).jpg
categories: [DevOps, Kubernetes,linux,Linux,Server,Deploy,Deployment]
---

![iasbdua](/assets/images/redhat1/Untitled design (4).jpg)

### Overview 
pada postingan kali ini saya akan membahas eror-eror yang di temui saat melakukan installasi kubernetes di rhel,karena saya pertama kali menginstall
kubernetes di rhel jadinya ada beberapa eror yang saya alami maka dari itu saya akan jadikan sebuah pembelajaran, langsung saja masuk ke pembahasannya


### Interface tertentu tidak mendapatkan ip 

ada momen dimana interface dari salah satu interface yang sudah di setup sebelumnya tidak mendapatkan ip,itu terjadi karena interface tersebut
belum up jadi tidak ada connection up yang ada di interface tersebuh,jadinya interface tidak mendapatkan ip,biasanya ini terjadi di interface berjenis 
host only adapter

### Solve

nah karena si connectionnya belum up jadi sekalian aja tambahin gunakan command

```
sudo nmcli connection add type ethernet ifname enp0s8 con-name enp0s8
```

lalu jika sudah

```
sudo nmcli connection up ensp08
```

tinggal di check

### Error Devices Allow 

Untuk eror selanjutnya yaitu eror device allow ini berhubungan dengan konfigurasi dari cgroup dan kernel yang ada di virtualbox
dikarenakan di postingan yang [ini](https://rizkiyusupk.github.io/devops/kubernetes/linux/server/deploy/deployment/build-kubernetes-in-redhat/) 
nah dan juga untuk container runtime interface menggunakan crio yang dimana environment dari virtualbox tidak mendukung adanya crio,mulai dari
cgroup rules ataupun kernel level configuration jadinya crio tidak bisa berjalan dengan baik di virtual box maka dari itu tools yang ada di gunakan pun berbeda dengan postingan yang lain 

### Solve

Untuk menyelesaikan masalah ini cukup sederhana tinggal ganti dari virtual box ke vmware bisa langsung cari saja vmware,atau [klik link ini](https://www.techspot.com/downloads/189-vmware-workstation-for-windows.html#google_vignette)
sudah selesai deh 

