---
title: "Build Integrated Deployment System Full Setup"
date: 2025-08-16
tags: [kubernetes, vm, virtualbox, containerd, flannel, cluster, devops,system,jenkins,Jenkins]
header:
  teaser: /assets/images/ci-cd/Untitled design (2).png
categories: [DevOps, Kubernetes]
---

![Kubernetes Cluster Architecture](/assets/images/ci-cd/Untitled design (2).png)

##  **Overview**
Saya akan membangun Kubernetes cluster production-like dengan 1 Master,2 Worker dan 1 jenkins Node menggunakan VirtualBox. Setup ini cocok untuk development, testing, dan pembelajaran Kubernetes.Semua ini bertujuan untuk pembelajaran dan pemahaman semata agar saya dan yang melihat ini bisa paham dalam secara mendalam apa itu Ci/Cd dan praktik di industri Ci/Cd,

##  **Spesifikasi Hardware & Software**

### **Virtual Machine Requirements:**

| Node        | CPU     | RAM  | Storage | Network                     |
|-------------|---------|------|---------|------------------------------|
| **Master**  | 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Host-only) |
| **Worker 1**| 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Host-only) |
| **Worker 2**| 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Host-only) |
| **Jenkins** | 7 cores | 7GB  | 30GB    | 2 Adapters (NAT + Host-only) |

berikut topologi yang saya pakai 
![logo3](/assets/images/ci-cd/WhatsApp Image 2025-11-16 at 20.37.31 (2).jpeg)
dan
![logo4](/assets/images/ci-cd/WhatsApp Image 2025-11-16 at 20.37.31 (3).jpeg)
### **Software Versions:**
- **OS**: Ubuntu Server 24.04 LTS
- **Kubernetes**: v1.28.15
- **Container Runtime**: containerd v1.27.7
- **CNI Plugin**: Flannel
- **VirtualBox**: 7.0.10
- **Jenkins**: 2.528.2
- **Java Open-jdk**: 17.0.17
-  **Ngrok**: 3.33.1


##  **Step 1: Kubernetes Setup**
**NOTE UNTUK SETUP KUBERNETES INI SAMA DENGAN SETUP KUBE DI POSTINGAN YANG SEBELUMNYA JADI INI SAMA TIDAKA DA BEDANYA UNTUK SETUP SI KUBERNETESNYA**
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
karena ini emang untuk bahan pembelajaran selanjutnya,**note karena ini menenggunakan preallocate full size sebaiknya gunakan disk yang lebih kecil 
bertujuan agar nantinya tidak ada beban disistem karena ini hanya untuk lab pembelajaran semata!!!!!**

![vm3](/assets/images/kube_cluster/kontol.png)

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


##Setup Kubernetes
---

Karena kamu sudah berhasil dengan installasi tadi sekarang masu ke tahap setup kubernetes
untuk langkah pertama kamu bisa langsung akses ke vm atau seperti yang saya akan tunjukan lewat ssh di terminal
klik **windows + r** lalu ketik **cmd** setelah itu tekan enter dengan begitu akan muncul jendela terminal

![vm14](/assets/images/kube_cluster/Screenshot 2025-08-17 232944.png)

lalu setelah itu kamu mengconfigurasi semua alamat ip dan nama hostname dari semua vm mu

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.


```
sudo vim /etc/hosts
```

lalu didalam file konfigurasinya kamu isi dengan

```
192.168.xxx.xxx nama_hostname_vm_mu
192.168.xxx.xxx nama_hostname_vm_mu
192.168.xxx.xxx nama_hostname_vm_mu
```
maka hasil nya akan seperti contoh ini

![vm17](/assets/images/kube_cluster/Screenshot 2025-08-18 003936.png)

setelah itu menonaktifkan firewall dan port yang akan digunakan,[apa itu firewall](https://tuxcare.com/blog/linux-firewalls/#:~:text=Linux%20Firewalls%20are%20the%20first,Linux%20are%20iptables%20and%20nftables.) dan [apa itu port](https://www.cloudflare.com/learning/network-layer/what-is-a-computer-port/#:~:text=Ports%20are%20virtual%20places%20within,the%20network%20traffic%20they%20receive.),semua port yang di butuhkan akan di configurasikan disini

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
#semua port yang dibutuhkan di master
sudo ufw allow "OpenSSH"
sudo ufw enable
sudo ufw allow 6443/tcp
sudo ufw allow 2379:2380/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10259/tcp
sudo ufw allow 10257/tcp
```
dan untuk vm worker

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp
```

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo apt-get update
```

setelah mengupdate package repository tahap selanjutnya ialah menonaktifkan swapp yang ada di ubuntu server mu
jika kamu ingin melihat nya coba ketik command ini 

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
free -h
```

coba lihat di bagian terminalnya kamu bisa lihat ada kolom **free** nah itu yang akan di nonaktifkan
karena kubelet tidak akan berjalan jika ada swapp yang on

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
#ini itu untuk menonaktifkan swapnya tapi sementara
sudo swapoff -a

#dan masukan command ini untuk menonaktifkannya secara permanen
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
nah selanjutnya ialah optimasi disk i/o agar tidak terlalu berat

```
cat <<EOF | sudo tee -a /etc/sysctl.d/k8s.conf
vm.dirty_ratio = 40
vm.dirty_background_ratio = 10
vm.swappiness = 1
EOF

sudo sysctl --system

```
jika kamu ingin tahu kenapa swap harus off ketika ingin install kubernetes [lihat disini](https://stackoverflow.com/questions/40553541/disable-swap-on-a-kubelet?utm_source=chatgpt.com)
setelah swap dimatikan kamu bisa langsung ke tahap selanjutnya yaitu mennconfigurasi modul untuk cri 
[apa itu cri](https://kubernetes.io/docs/concepts/containers/cri/#:~:text=The%20Container%20Runtime%20Interface%20CRI,components%20kubelet%20and%20container%20runtime)
tahap ini berfungsi untuk menulis modul-modul secara presistent agar nantinya bisa mendukung komunikasi dan building layer untuk container

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
# Load modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Buat persistent
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

module diatas berfungsi untuk mendukung layering saat pembuatan container nantinya dan juga untuk bridging
trus untuk membuat configurasinya persistent ditambahkan juga ke **/etc/modules-load.d/k8s.conf** nantinya modul akan tetap ada walaupun sistem di reboot
nah selanjutnya ialah configurasi untk iptables dan juga untuk ipv4

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.


```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl
sudo sysctl --system
```

tahap selanjutnya ialah installasi untuk cri (container runtime interface) kita disini akan menggunakan containerd

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
# Install dependencies
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository (Ubuntu 24.04 menggunakan noble)
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install containerd
sudo apt update
sudo apt install -y containerd.io

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Edit config untuk systemd cgroup (Ubuntu 24.04 sudah default systemd)
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd
```

jika sudah bisa kita langsung ke tahap selanjutnya

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
# Add Kubernetes GPG key dan repository
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install Kubernetes tools
sudo apt update
sudo apt install -y kubelet kubeadm kubectl

# Hold packages dari auto-update
sudo apt-mark hold kubelet kubeadm kubectl

# Enable kubelet
sudo systemctl enable kubelet
```
**lakukan semua step diatas disemua vm!!!!**
**SELANJUTNYA INIT DI VM MASTER**
untuk step selanjutnya ialah init dari vm master

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo kubeadm init \
  --apiserver-advertise-address=192.168.xxx.xxx \
  --pod-network-cidr=10.244.0.0/16 \
  --node-name=nama-vm-mastermu
```

tunggu beberapa saat memang memakan waktu yang cukup lama,jika sudah akan ada output yang perlu dicatat 

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
copy semua lalu paste di terminal untuk membuat direktori config dari kube,setelah itu di simpan kamu bisa langsung join dengan token output yang 
muncul bersamaan dengan command tadi
nah selanjutnya ialah tahap instalasi untuk network disini saya menggunakan network flannel

```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
lalu setelah itu bisa langsung join menggunakan output dari init tadi 

```
sudo kubeadm join 192.168.xxx.xxx:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

cari output itu diantara output **sudo kubedm init** tadi pastinya sudah sesuai dengan vm master mu,setelah itu copy lalu paste di vm worker 
verifikasi ✔️✔️✔️✔️✔️✔️✔️✔️✔️✔️


![vm16](/assets/images/Screenshot 2025-08-17 232944.png)

setelah itu selesai sudah semua config 🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀

## **Step 2: Jenkins Setup**
Pada tahap ini saya akan membuat kembali 1 vm baru untuk si jenkinsnya,sebenarnya bisa saja saya membuat versi containernya yang bisa di scale dikarenakan saya itu masih belajar untuk versi containernya akan saya tunda terlebih dahulu,untuk stepnya sama seperti membuat vm kube sebelumnya jadi bisa langsung skip ke installasi jenkinsnya
pertama-tama update lalu buat install openjdk jika java belum terinstall,kenapa butuh java karena jenkins tidak akan berjalan tanpa java 
[selengkapnya](https://www.jenkins.io/solutions/java/#:~:text=Jenkins%20supports%20building%20Java%20projects,the%20tool%20many%20years%20ago.)
,kita skip langsung ke instalasinya update package terlebih dahulu

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo apt-get update
```

jika sudah bisa langsung install openjdknya 

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo apt install openjdk-17-jdk -y
```

jika sudah pastikan sudah terinstall untuk javanya

```
java --version
```

nah untuk outputnya seperti ini 
![logo3](/assets/images/ci-cd/Screenshot 2025-12-07 202907.png)

jika outputnya sudah seperti itu maka selamat java sudah terinstall di vmnya,next selanjutnya ialah instalasi jenkins karena java sudah terinstall maka untuk selanjutnya ialah jenkins,kita bisa langsung masuk ke installasinya 

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

```

tambahkan repo jenkins resmi,setelah itu 

```
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

lalu install jika sudah menambahkan semua kebutuhannya

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo apt update
sudo apt install jenkins
```

jika sudah dan tidak ada eror satupun itu berarti installasi berhasil dan cek apakah benar-benar berhasil dengan

```
jenkins --version
```

outputnya seharunya seperti ini

![logo4](/assets/images/ci-cd/Screenshot 2025-12-07 205215.png)

jika sudah sekarang bisa langsung ke tahap selanjutnya yaitu 
