---
title: "Pull Based Infrastructure Hub & Spoke "
date: 2026-10-22
infra_used: "Pull Based Argocd"
---

![asdjadvpisfb](/assets/images/hub_and_spoke/hb.png)

### Overview

Kali ini saya akan membuat sebuah case page dimana saya akan mengotak otak sebuah infrastructure yang sudah saya buat sebelumnya,nah kali ini saya 
akan mengotak atik infra dari pull based yang sudah saya buat dan sedikit mengubahnya,jika di projek utama infra pull based semua cluster memiliki 
satu argocd bisa di sebut bahwa infra itu stand alone argocd nah tapi untuk yang kali ini hanya akan ada satu argocd di dua cluster dan infra ini
disebut dengan hub and spoke infrastructure,dan untuk hub akan di cluster jakarta dan untuk spokenya akan di cluster bandung,oke tidak usah berlama-
lama langsung saja masuk ke pembahasannya

| Node        | CPU     | RAM  | Storage | Network                                             |
|-------------|---------|------|---------|---------------------------------------------------- |
| **Master-cluster-Jakarta**  | 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 1-cluster-Jakarta**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 2-cluster-Jakarta**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Master-cluster-Bandung**  | 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 1-cluster-Bandung**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 2-Cluster-Bandung**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Jenkins**                 | 7 cores | 7GB  | 240GB   |                Wlan                 |

### Structure Folder

Untuk structure folder akan terdiri dari dua folder yang pertama untuk terraform 

```
terraform-setup/
├── .terraform/
├── compute.tf
├── main.tf
├── prep-vm.tf
├── prep-2.tf
├── compute-cluster-2.tf
```

dan yang kedua yaitu ansible

```
k8s/
├── ansible.cfg
├── inventory
├── playbook-allow-port.yaml
├── playbook-ip.yaml
├── playbook-join.yaml
├── playbook-kubernetes.yaml
├── playbook-pkg.yaml
├── playbook-swap.yaml
├── playbook-config.yaml
├── argocd.yaml
```

### Reasoning
Kenapa saya memutuskan untuk membuat hub and spoke infra?? karena saya melihat bahwa infra stand alone itu tidak tersentralisasi atau terpusat 
pada satu cluster dan kurangnya aspek keamanan atau security aspect,dan juga gampang untuk di scale up jika saja ada cluster baru 

### Setup 
Oke untuk langkah pertama cek apakah masih ada application di dalam argocd  di cluster bandung,jika ada hapus terlebih dahulu,lalu untuk langkah selanjutnya bisa di pilih 
apakah mau di uninstall atau tidak untuk argocd cluster bandung,meskipun argocd cluster bandung masih ada dan tidak di hapus tidak apa apa tidak berpengaruh,karena
application yang ada di dashboard ui argocd bandung sudah di hapus,jadinya hanya akan ada satu argocd yang akan mengarah ke app config bandung yang di projek infra awal
sebelumnya di buat,oke jadi yang harus di pastikan di tahap ini adalah tidak ada application di dashboard argocd  2
