---
title: "Kubernetes Cluster dengan 1 Master dan 2 Worker menggunakan 2 VM"
date: 2025-08-16
tags: [kubernetes, vm, containerd, flannel, cluster]
header:
  teaser: /assets/images/images.png

---
![kubelogo](/assets/images/images.png)

## 📖 Apa itu Kubernetes?
Kubernetes (sering disingkat **k8s**) adalah sebuah **platform orkestrasi container** yang digunakan untuk mengelola aplikasi berbasis container secara otomatis. Dengan Kubernetes, kita bisa:

- Melakukan **deployment aplikasi** dengan mudah.  
- Melakukan **scaling** (naik/turun) sesuai kebutuhan.  
- Mengatur **load balancing** antar service.  
- Memberikan **self-healing**, yaitu pod yang gagal otomatis dijalankan ulang.  
- Menyediakan **networking dan service discovery** antar container.  

Singkatnya, Kubernetes memungkinkan kita untuk mengelola cluster server sebagai satu kesatuan sistem.



## 🖥️ Apa itu Virtual Machine (VM)?
**Virtual Machine (VM)** adalah sebuah **mesin komputer virtual** yang berjalan di atas hardware fisik melalui software hypervisor seperti VirtualBox, VMware, atau KVM.  

VM memiliki resource sendiri (CPU, RAM, Storage) yang dialokasikan dari host.  
Dalam konteks Kubernetes, VM sering digunakan untuk membuat cluster secara lokal atau dalam lab environment sebelum ke server produksi.



## ⚙️ Tools Yang Akan Digunakan
Berikut Datail Hal-hal yang akan digunakan:

- **Sistem Operasi**: Ubuntu 24.04 LTS (ubuntu sever iso).
- **Virtual Box Versi:** `1.7.10`
- **Container Runtime**: `containerd v1.27.7`.  
- **Kubernetes Tools**:  
  - `kubeadm v1.28`  
  - `kubectl v1.28`  
  - `kubelet v1.28`  
- **CNI Plugin (Networking)**: Flannel.  
- **Cluster Topologi**:  
  - 1 Master Node  
  - 2 Worker Node  
test


## Install Virtual Machine (VM)
---
Pertama-tama buat topologi dari cluster terlebih dahulu yaitu 1 master 2 worker,dengan ini
kita perlu 3 **Virtual Machine** (**VM**),[selengkapnya mengenai installasi vm dan apa itu vm](https://www.virtualbox.org/),
disini saya akan menggunakan iso **ubuntu 24** [selengkapnya untuk iso ubuntu 24](https://ubuntu.com/download/server),setelah kamu mendownload iso ubuntu 
sekarang kamu tinggal melakukan installasi untuk vmnya.
Pertama-tama kamu masuk terlebih dahulu ke menu dashboard dari virtual box

![vm](/assets/images/kube_cluster/Screenshot 2025-08-16 212949.png)

Terus lakukan seperti yang ada digambar,setelah kamu masuk ke menu **machine**  lalu klik tombol **new**
setelah itu kamu akan diarahkan ke menu seperti ini

![logovm1](/assets/images/kube_cluster/Screenshot 2025-08-16 213025.png)

kamu isi dengan nama yang kamu inginkan,lalu setelah itu pilih linux dengan subtype ubuntu pilih saja ubuntu x64
setelah itu klik bagian Hardware dan pilih 12288 untuk memory dan 8 cpu karena ini adalah sebuah cluster dari kubernetes yang nantinya
akan digunakan di waktu yang akan mendatang untuk belajar maka persiapkan resource yang memadai

![vm2](/assets/images/kube_cluster/Screenshot 2025-08-16 213042.png)

jika sudah maka akan terlihat seperti itu,setelah itu masuk ke bagian Hardisk,lalu set size sebesar 130gb
karena ini emang untuk bahan pembelajaran selanjutnya

![vm3](/assets/images/kube_cluster/Screenshot 2025-08-16 213053.png)

jika sudah,klik finish setelah itu kamu klik kanan pada bagian machine lalu klik settings

![vm3](/assets/images/kube_cluster/Screenshot 2025-08-16 213112.png)

setelah itu bakalan diarahkan ke menu general lalu masuk ke menu network,lalu kamu masuk ke menu adapter 2

![vm1](/assets/images/kube_cluster/Screenshot 2025-08-17 195628.png)

setelah kamu masuk ke menu adapter 2 klik tombol enable 
setelah kamu enable adapter networknya pilih host-only adapter

![vm](/assets/images/kube_cluster/Screenshot 2025-08-17 195638.png)

