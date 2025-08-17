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

jangan edit adapter 1 biarkan default saja,setelah itu klik ok lalu klik pada bagian
storage dan pilih bagian optical drive itu ada file iso yang akan digunakan,kamu bisa pilih search di file download dengan mengklik choose a disk file

![vm4](/assets/images/kube_cluster/memek.png)

setelah kamu memilih file di disk berikutnya tinggal start si machine 
setelah start beberapa saat kemudian akan muncul opsi **try or install ubuntu** pencet enter pada bagian itu
lalu kamu akan dibawa ke menu pemilihan bahasa 

![vm1](/assets/images/kube_cluster/VirtualBox_Master1_16_08_2025_21_53_33.png)

pilih bahasa yang kamu sukai saya disini memilih bahasa inggris,jika sudah tekan enter

![vm4](/assets/images/kube_cluster/VirtualBox_Master1_16_08_2025_21_53_51.png)

maka akan di arahkan ke menu selanjutnya,jika terdapat menu seperti ini 

![vm5](/assets/images/kube_cluster/VirtualBox_Master1_16_08_2025_21_53_51.png)

maka tekan enter saja untuk default,lalu selanjutnya untuk opsi ini

![vm5](/assets/images/kube_cluster/VirtualBox_Master1_16_08_2025_21_54_04.png)

langsung tekan enter karena kita akan memakai full ubuntu server,setelah itu akan muncul opsi untuk network yang akan digunakan

![vm6](/assets/images/kube_cluster/VirtualBox_Master1_16_08_2025_22_16_19.png)

langsung tekan enter saja untuk default jangan mengotak-atik apapun,

![vm7](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_22_59_06.png)

setelah kamu mengklik enter maka selanjutnya akan ada opsi untuk mengconfigurasi proxy address

![vm7](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_22_59_06.png)

langsung klik enter saja disini kita belum mengconfigurasi proxy address,nah selanjutnya kita akan dibawa ke
opsi untuk mengconfigurasi storage

![vm8](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_22_59_42.png)

disini disable pilihan untuk lvm dengan cara tekan space pada opsi kotak lvm 

![vm9](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_22_59_42.png)

setelah itu pilih done lalu enter,selanjutnya akan ada opsi agar kamu bisa mengulang configurasimu jika masih ragu bisa pilih 
**reset atau back** lalu tekan enter,jika dirasa sudah pas pilih opsi **done** lalu continue

![vm9](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_22_59_57.png)

setelah itu kamu akan dibawa ke configurasi selanjutnya,disini kamu akan diminta mengisi profile untuk server seperti nama,nama server,username,dan password

![vm10](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_22_59_57.png)

jika sudah mengisi semua field bisa langsung pilih done lalu enter,jika ada opsi seperti ini bisa langsung pilih done lalu enter

![vm11](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_23_00_33.png)

setelah itu kamu bisa menginstall ssh dengan menekan space untuk opsi install ssh,ssh ini berguna untuk nanti nya bisa remote si server 
[berikut detail mengenai ssh](https://www.1kosmos.com/security-glossary/secure-shell-ssh/#:~:text=Secure%20Shell%20(SSH)%20is%20a,manage%20remote%20systems%20and%20servers.)

![vm12](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_23_00_46.png)

setelah menginstall ssh selanjutnya kamu bisa memilih opsi untuk install applikasi lainnya seperti docker containerd,dll
namu disini kita tidak akan memilih opsi opsi yang tadi langsung saja pakai kursor bawah sampai opsi done,karena jika kita memilih opsinya maka installasi akan
dilakukan menggunakan **snap** [soal snap](https://ubuntu.com/tutorials/create-your-first-snap#1-overview),karena kita tidak akan memakai snap untuk installasi melainkan lewat
**apt** atau package manager ubuntu,[referensi apt](https://ubuntu.com/tutorials/create-your-first-snap#1-overview),setelah itu kamu bisa langsung pilih opsi done lalu tekan enter

![vm13](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_23_01_07.png)

setelah itu kamu bisa tunggu untuk proses installasinya selesai!!

![vm15](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_23_03_41.png)

jika sudah tinggal di reboot lalu installasi seesai 🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀
