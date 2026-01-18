---
title: "Build Ci/Cd Hybrid System Full Infrastructure Setup(Vm + Bare Metal)"
date: 2026-01-17
tags: [kubernetes, linux, cli, devops, containerd,bare-metal,hybrid,ci,cd,ci/cd,system,infrastructure]
header:
  teaser: /assets/images/hybird-ci-cd/Untitled design (1).jpg
categories: [DevOps, Kubernetes,linux,Linux,Server]
---

![logo1](/assets/images/hybird-ci-cd/jembod.jpg)

## Overview 

Pada post kali ini saya akan mendemonstrasikan step by step setup infrastruktur dari sebuah sistem ci/cd,
sama seperti di post sebelumnya soal yang [ini](https://rizkiyusupk.github.io/devops/kubernetes/Build-Ci-Cd-System/)
di post ini perbedaanya hanyalah pada topologi,jika di post sebelumnya itu full vm dipost kali ini saya akan melakukannya 
dengan tambahan 1 bare metal yaitu sebuah laptop.Bare metal disini itu berfungsi sebagai tambahan pembalajaran dan 
menambah kerumitan dari topologi sebelumnya, yang akan di gantikan bare metal yaitu vm jenkins yang sebelumnya berupa sebuah
vm di gantikan menjadi bare metal.Pada pratiknya ini nantinya akan melibatkan banyak sekali tools-tools dan beberapa,untuk topologi masih 
sama dengan yang lama cuman ada beberapa koreksi mengenai topologi seperti penambahan bare metal jenkins.Sementara untuk workflow itu
masih sama dengan yang sebelumnya jadi,cara kerja atau workflownya sama dengan post sebelumnya.
Berikut workflow dari project ini

![logoo1](/assets/images/ci-cd/2.png)

untuk topologinya itu akan ada sedikit perubahan jadi seperti ini 

![logoo1](/assets/images/hybird-ci-cd/d2de8764-f43f-4152-ba17-0de4a26c8166.jpg)

untuk spesifikasi vm
### **Virtual Machine Requirements:**

| Node        | CPU     | RAM  | Storage | Network                            |
|-------------|---------|------|---------|------------------------------------|
| **Master**  | 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Bridged-Adapter) |
| **Worker 1**| 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Bridged-Adapter) |
| **Worker 2**| 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Bridged-Adapter) |
| **Jenkins** | 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Bridged-Adapter) |

kenapa saya di projek ini merubah dari yang tadinya host-only menjadi bridged-adapter?
karena saya di projek ini menggunakan bare metal tambahan jadinya saya perlu satu koneksi yang sama
diantara bare metal dan vm,jika saya tetap menggunakan host-only adapter nantinya subnet antara vm dan 
bare metal akan berbeda jadinya tidak akan saling terhubung,
dan untuk tools yang akan di pakai itu seperti

### **Software Versions:**
- **OS**: Ubuntu Server 24.04 LTS (Vm Kubernetes) 
- **OS**: Ubuntu 25.04 (Bare Metal / Jenkins )
- **Kubernetes**: v1.28.15
- **Container Runtime**: containerd v2 2.1.3
- **CNI Plugin**: Flannel
- **VirtualBox**: 7.0.10
- **Jenkins**: 2.528.2
- **Java Open-jdk**: 17.0.17
- **Ngrok**: 3.35.0
- **Git Bash** : git version 2.51.1.windows.1

##  **Step 1: Kubernetes Setup**
**NOTE UNTUK SETUP KUBERNETES INI SAMA DENGAN SETUP KUBE DI POSTINGAN YANG SEBELUMNYA JADI INI SAMA TIDAKA DA BEDANYA UNTUK SETUP SI KUBERNETESNYA**
Pertama-tama buat topologi dari cluster terlebih dahulu yaitu 1 master 2 worker,dengan ini
kita perlu 3 **Virtual Machine** (**VM**),[selengkapnya mengenai installasi vm dan apa itu vm](https://www.virtualbox.org/),
disini saya akan menggunakan iso **ubuntu 24** [selengkapnya untuk iso ubuntu 24](https://ubuntu.com/download/server),setelah kamu mendownload iso ubuntu 
sekarang kamu tinggal melakukan installasi untuk vmnya.
Pertama-tama kamu masuk terlebih dahulu ke menu dashboard dari virtual box

![vm](/assets/images/Screenshot 2026-01-07 211210.png)

Terus lakukan seperti yang ada digambar,setelah kamu masuk ke menu **machine**  lalu klik tombol **new**
setelah itu kamu akan diarahkan ke menu seperti ini

![logovm1](/assets/images/Screenshot 2026-01-07 211225.png)

kamu isi dengan nama yang kamu inginkan,lalu setelah itu pilih linux dengan subtype ubuntu pilih saja ubuntu x64
setelah itu klik bagian Hardware dan pilih 7168 untuk memory dan 7 cpu karena ini adalah sebuah cluster dari kubernetes yang nantinya
akan digunakan di waktu yang akan mendatang untuk belajar maka persiapkan resource yang memadai

![vm2](/assets/images/Screenshot 2026-01-07 211225.png)

jika sudah maka akan terlihat seperti itu,setelah itu masuk ke bagian Hardisk,lalu set size sebesar 30gb
karena ini emang untuk bahan pembelajaran selanjutnya,karena ditutorial ini itu memang full size 30 gb 1 vm

![vm3](/assets/images/Screenshot 2026-01-07 211246.png)

jika sudah,klik finish setelah itu kamu klik kanan pada bagian machine lalu klik settings

![vm3](/assets/images/Screenshot 2026-01-07 211331.png)

setelah itu bakalan diarahkan ke menu general lalu masuk ke menu network,lalu kamu masuk ke menu adapter 2

![vm1](/assets/images/Screenshot 2026-01-07 211341.png)

setelah kamu masuk ke menu adapter 2 klik tombol enable 
setelah kamu enable adapter networknya pilih bridged adapter

![vm](/assets/images/Screenshot 2026-01-07 211358.png)

dan untuk promiscuous modenya pilih allow all
jangan edit adapter 1 biarkan default saja,setelah itu klik ok lalu klik pada bagian
storage dan pilih bagian optical drive itu ada file iso yang akan digunakan,kamu bisa pilih search di file download dengan mengklik choose a disk file

![vm4](/assets/images/Screenshot 2026-01-07 211416.png)

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

![vm10](/assets/images/kube_cluster/VirtualBox_master1_16_08_2025_23_00_08.png)

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
**jika sudah kamu tinggal mengulangi hal yang sama,dan jangan lupa untuk mengganti nama dari vmnya dengan worker** 🥳🥳🥳🥳🥳🥳🥳🥳

