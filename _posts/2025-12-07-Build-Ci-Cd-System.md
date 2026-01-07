---
title: "Build Integrated Deployment System Full Setup"
date: 2025-12-11
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
![logo3](/assets/images/ci-cd/2.png)
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
- **Ngrok**: 3.33.1
- **Git Bash**


##  **Step 1: Kubernetes Setup**
**NOTE UNTUK SETUP KUBERNETES INI SAMA DENGAN SETUP KUBE DI POSTINGAN YANG SEBELUMNYA JADI INI SAMA TIDAKA DA BEDANYA UNTUK SETUP SI KUBERNETESNYA**
Pertama-tama buat topologi dari cluster terlebih dahulu yaitu 1 master 2 worker,dengan ini
kita perlu 3 **Virtual Machine** (**VM**),[selengkapnya mengenai installasi vm dan apa itu vm](https://www.virtualbox.org/),
disini saya akan menggunakan iso **ubuntu 24** [selengkapnya untuk iso ubuntu 24](https://ubuntu.com/download/server),setelah kamu mendownload iso ubuntu 
sekarang kamu tinggal melakukan installasi untuk vmnya.
Pertama-tama kamu masuk terlebih dahulu ke menu dashboard dari virtual box

**NOTE GAMBAR ADA SEDIKTI KESALAH MENGENAI RUANG SAYA BELUM MEMPERBAIKINYA GAMBAR INSTALLASI VM TIDAK SESUAI DENGAN SPESIFIKASI!!ABAIKAN SAJA**
**NOTE INSTALLASI TIDAK HARUS MENGIKUTI SPESIFIKASI YANG ADA DI GAMBAR JADI IKUTI SAJA SESUAI BACAA NANTI SAYA AKAN PERBAIKI BEBERAPA GAMBAR
YANG MEMANG ADA YANG RUSAK DAN TIDAK SESUAI TERIMAKASIH**

![vm](/assets/images/Screenshot 2026-01-07 211210.png)

Terus lakukan seperti yang ada digambar,setelah kamu masuk ke menu **machine**  lalu klik tombol **new**
setelah itu kamu akan diarahkan ke menu seperti ini

![logovm1](/assets/images/Screenshot 2026-01-07 211225.png)

kamu isi dengan nama yang kamu inginkan,lalu setelah itu pilih linux dengan subtype ubuntu pilih saja ubuntu x64
setelah itu klik bagian Hardware dan pilih 7168 untuk memory dan 7 cpu karena ini adalah sebuah cluster dari kubernetes yang nantinya
akan digunakan di waktu yang akan mendatang untuk belajar maka persiapkan resource yang memadai

![vm2](/assets/images/Screenshot 2026-01-07 211225.png)

jika sudah maka akan terlihat seperti itu,setelah itu masuk ke bagian Hardisk,lalu set size sebesar 30gb
karena ini emang untuk bahan pembelajaran selanjutnya,**note karena ini menenggunakan preallocate full size sebaiknya gunakan disk yang lebih kecil 
bertujuan agar nantinya tidak ada beban disistem karena ini hanya untuk lab pembelajaran semata!!!!!**

![vm3](/assets/images/Screenshot 2026-01-07 211235.png

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


![vm16](/_posts/2025-12-07-Build-Ci-Cd-System.md)

diatas merupakan contoh yang saya ambil dari internet,diatas merupakan sebuah contoh utuk output join command,jika sudah semua maka
kubernetes berhasil di buat

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

jika sudah sekarang bisa mulai jenkins dengan menggunakan command

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

check status jenkinsnya dengan 

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo systemctl status jenkins
```

sekarang masuk tahap yang krusial tahap dimana installasi untuk kubectl,kenapa cuman kubectl engga sekalian aja kubeadm,kubelet? dikarenakan hanya
kubectl yang akan berguna dan terpakai jadi pada saat penggunaan hanya kubectl saja yang akan mengakses ke clusternya tidak dengan kubeadm maupun kubelet,maka dari itu hanya kubectl yang akan di install,langsung saja tambahkan dependencies

```
sudo apt install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

setelah itu 
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

setelah semua kebutuhan untuk installasi kubectl selesai bisa langsung masuk tahap installasi saja

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo apt-get update
sudo apt install kubectl -y
```

jika proses sudah berhasil dan tidak ada eror pastikan apakah kubectl terinstall dengan command

```
kubectl version --client
```

outputnya akan seperti ini
![logo1](/assets/images/ci-cd/Screenshot 2025-12-07 225158.png)

jika outputnya sudah sesuai itu berarti kubectl terinstall dengan benar untuk tahap selanjutnya itu copy kubeconfig dari master ke jenkins vm
saya akan melakukannya lewat terminal ssh,buka cmd di windows dengan menekan tombol 

```
windows + r
lalu ketik cmd+enter
```

jika sudah maka kamu akan masuk ke tampilan terminal,ssh ke vm kube master dengan command

```
ssh host@ip-vm
misal
ssh rizki@192.168.56.167
```

jika sudah masuk ke terminal master ketik command

```
cat ~/.kube/config
```

output seharusnya seperti ini 

![logo7](/assets/images/ci-cd/Screenshot 2025-12-07 225732.png)

copy semua hasil dari output itu lalu ssh ke terminal jenkins dengan cara yang sama seperti ke kube master
jika sudah buat sebuah direktori untuk meyimpan file user jenkins

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo mkdir -p ~/.kube/
```

jika sudah copy output tadi di masukan ke file di dalam direktori yang baru saja di buat

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo vim ~/.kube/config
```

setelah itu ubah perizinan dari si filenya 

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo chown -R $USER:$USER ~/.kube
```

jika sudah di paste test untuk uji coba akses ke kube gunakan command 

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo -u jenkins kubectl get nodes
```

untuk test saja gunakan command diatas,output yang akan di keluar itu seperti ini

![logo3](/assets/images/ci-cd/Screenshot 2025-12-07 232527.png)

jika sudah keluar output nodesnya berarti sekarang jenkins bisa akses ke kubernetes cluster 

## SSH SETUP
untuk tahap selanjutnya generate ssh key 
untuk commandnya seperti ini

```
ssh-keygen -t rsa
```

tekan enter terus sampai proses selesai agar ssh key dalam mode default,lakukan lagi di cluster kube,jika sudah 
melakukannya ke semua node copy ssh-keygen ke semua node menggunakan command

```
ssh-copy -i -i /PATH/TO/FILE host@ip-vm
atau
ssh-copy- -i /home/rizki/.ssh/pub rizki@192.168.56.167
```

copy command diatas itu akan mengcopy dari vm jenkins ke master,tekan tab pada saat setelah mengetik flag -i agar mempercepat proses
lakukan kesemua vm

```
jenkins -> master
jenkins -> worker
jenkins -> worker1
---
master -> jenkins
master -> worker
master -> worker1
---
worker -> master
worker -> jenkins
worker -> worker1
---
worker1 -> master
worker1 -> jenkins
worker1 -> worker

```

lakukan semua step diatas ke masing masing nodes dengan command yang sudah di tunjukan 

## SETUP GITLAB
Pada tahap ini saya akan mensetup atau configurasi gitlab untuk nantinya bisa terhubung dengan node,
pertama-tama buat keybaru lalu simpan di direktori user yang sebelumnya sudah di buat 

```
sudo -u jenkins ssh-keygen -t rsa -b 4096 -C "jenkins@your-vm" -f /var/lib/jenkins/.ssh/id_rsa -N ""
```

jika ingin copy dari ssh-key sebelumnya juga tidak masalah atau malah ingin menimpa dengan yang ini tapi jika ingin menimpa maka
harus mencopy ulang langkah sebelumnya,jika sudah buka id_rsa.pub file yang tersimpan lalu copy 

![logo12](/assets/images/ci-cd/Screenshot 2025-12-08 183743.png)

kurang lebih seperti ini nah jika sudah lalu copy semua outputnya lalu pergi ke browser lalu ketik gitlab,pastikan punya akun gitlabnya setelah 
itu login lalu pergi ke edit profile setelah itu 
masuk ke menu ssh-key di sebelah kiri

![logo13](/assets/images/ci-cd/Screenshot 2025-12-08 183650.png)

kurang lebih seperti diatas menu di bagian ssh-key,nah setelah itu klik add new key,paste yang sudah di copy tadi lalu 
tambahkan title misal auth lalu set ke authentication & signing setelah itu set expiration datenya bebas terserah kamu

![logo02](/assets/images/ci-cd/Screenshot 2025-12-08 183717.png)

setelah itu test apakah jenkins sudah terhubung dengan gitlab dengan command 

```
sudo -u jenkins ssh -T git@gitlab.com
```

kurang lebih outputnya seperti ini

![logo01](/assets/images/ci-cd/Screenshot 2025-12-08 190504.png)

jika output sama seperti diatas selamat karena gitlab sudah berhasil terhubung dengan jenkins

## CONFIG JENKINS
untuk step selanjutnya masuk ke tahap config jenkins buka browser mu lalu masuk ke ui dari jenkins dengan mengetikan 

```
ip-vm:8080
misal
192.168.56.169:8080
```

jika sudah maka akan ada tampilan seperti ini diawal

![logo](/assets/images/ci-cd/setup-jenkins-01-unlock-jenkins-page.jpg)

buka saja file yang berisi passwordnya jadi untuk masuk ke jenkins perlu password admin dan passwordnya ada di direktori defaultnya
lalu copy dan paste di kolom yang sudah disediakan,setelah itu akan diarahakan ke menu install plugin

![logo011](/assets/images/ci-cd/customize_jenkins_screen_two.png)

pilih suggested lalu akan diarahkan ke halaman seperti ini

![logo022](/assets/images/ci-cd/jenkins_plugin_install_two.png)

**NOTE** semua gambar diatas diambil dari internet jadi saya lupa untuk mengambil gambar dari proses installasi saya 
untuk step selanjutnya,buat secret di jenkins dengan masuk ke menu ui dari jenkins dan masuk ke dashboardnya setelah itu klik icon atau logo gear
disebelah kanan atas klik itu

![logoo](/assets/images/ci-cd/Screenshot 2025-12-08 234431.png)

jika sudah akan di bawa ke menu tampilan seperti dibawah ini,

![logooo](/assets/images/ci-cd/Screenshot 2025-12-08 234445.png)

jika sudah masuk ke menu settings cari credentials lalu klik menu credentials

![logoop](/assets/images/ci-cd/Screenshot 2025-12-08 234505.png)

diatas merupakan isi dari menu credentials,jika belum menambahkan credentials apapun klik domains global lalu add credentials

![logooi](/assets/images/ci-cd/Screenshot 2025-12-08 234646.png)

contohnya seperti diatas klik add credentials lalu kamu akan dibawa kemenu halaman seperti 

![logoou](/assets/images/ci-cd/Screenshot 2025-12-08 235703.png)

set credentials ke ssh userame with private key lalu di bagian private key klik enter directly lalu klik key dan setelah itu akan muncuk kolom dimana
kamu bisa menaruh ssh key yang sudah di generate sebelumnya pada saat setup gitlab,gunakan key itu lagi untuk di jenkins,username bebas dan id bebas itu terserah kamu
jika sudah save **NOTE** gambar diatas hanyalah contoh jadi key yang akans saya gunakan itu key yang berbeda dari contoh,

## SETUP NGROK
Dikarenakan gitlab yang saya pakai itu adalah gitlab web jadinya gitlab perlu mengakses ke ip vm saya sedangkan vm saya adalah local jadinya saya perlu sesuatu agar gitlab bisa mengakses vm saya maka saya akan menggunakan ngrok disini [install ngrok](https://ngrok.com/download/linux),jika sudah terinstall ambil token auth ngrok dengan masuk ke websitenya,jika sudah masuk dan login masuk ke your auth token 

![logoo1](/assets/images/ci-cd/Screenshot 2025-12-10 172759.png)

kurang lebih seperti ini tampilannya,copy lalu pergi ke terminal jenkins llau ketik command ini

```
ngrok config add-authtoken <token>
```

ganti token dengan auth token yang tadi di copy lalu jalankan ngrok dengan command

```
ngrok http 8080 
```

arahkan ke port dimana jenkins berjalan agar nantinya bisa melakukan webhook,jika sudah maka akan ada output seperti ini

![logoo9](/assets/images/Screenshot 2025-12-10 170011.png)

simpan link yang diberikan nantinya akan di pakai untuk webhook

## SETUP BASH
dikarenakan ini webhook jadi nantinya saya akan mendemokan bagaimana caranya sesuai dengan yang ada di gambar jadi ketika dev push dari laptop dev
ke gitlab nantinya gitlab akan menerima webhook event lalu triger jenkinsnya lalu jenkins akan mendeploynya ke kubernetes,jadi saya akan menstup bash terlebih dahulu,install dulu bash di laptop masing masing jika tidak ada [link bash](https://git-scm.com/install/windows),nah jika sudah masuk ke bashnya lalu login atau bisa langsung clone repo dengan cara

```
git clone url-repo
```

dengan begitu akan otomatis mengclone repo jika sudah masuk ke direktori repo lalu buat sebuah Jenkinsfile simple karena hanya untuk test webhook saja
berikut contoh Jenkinsnya

```
pipeline {
    agent any
    stages {
        stage('Test Webhook') {
            steps {
                echo 'Webhook triggered successfully!'
                sh 'ls -la'
            }
        }
    }
}
```

copy jenkins diatas dan paste kedalam file jenkins setelah itu simpan jangan dulu di masukan ke stagging mode simpan saja terlebih dahulu

## SETUP WEBHOOK 
jika setup bash sudah dilakukan kini masuk ketahap setup webhook masuk ke gitlab,masuk ke gitlab lalu masuk ke menu project dan 
klik New project

![logoou](/assets/images/ci-cd/Screenshot 2025-12-09 222155.png)

jika sudah maka akan di bawa ke tampilan halam seperti di bawah ini

![logoo8](/assets/images/ci-cd/Screenshot 2025-12-09 222207.png)

setelah itu klik create blank project,jika sudah maka akan dibawa ke halaman seperti ini

![logoo8p](/assets/images/ci-cd/Screenshot 2025-12-09 222226.png)

untuk tahap selanjutnya isi nama projectnya sesuai dengan keinginan lalu isikan project url dengan username gitlab kamu
pastikan project itu public jika tidak itu terserah kembali kepada diri masing-masing,setelah itu klik create project 
untuk tahap selanjutnya 

![logoo0](/assets/images/ci-cd/Screenshot 2025-12-09 222226.png)

seperti contoh diatas jika sudah masuk ke tahap selanjutnya untuk pembuatan pipeline,
masuk ke jenkins lalu klik new item pojok kanan atas lalu kamu bisa langsung memilih tipe jobsnya pilih pipeline lalu berinama

![najay](/assets/images/ci-cd/Screenshot 2025-12-09 223903.png)

seperti contoh diatas,jika sudah scroll ke bawah sampai bagian triggers jika sudah checklist bagian "Build when is pushed to gitlab "

![logoo1](/assets/images/ci-cd/Screenshot 2025-12-09 224117.png)

jika sudah kamu scroll ke bawah ke bagian advanced lalu kamu bisa generate secret token,ini penting sangat penting kamu copy dan simpan tokennya
nantinya token itu akan di pakai untuk webhook di gitlab,

![logoo112](/assets/images/ci-cd/Screenshot 2025-12-09 224128.png)

jika sudah scroll lagi kebawah sampai di menu pipeline definition lalu ubah tipe pipelinenya dari pipeline script ke 
ke pipeline scrip from scm 

![logooo1](/assets/images/ci-cd/Screenshot 2025-12-09 224136.png)

dari contoh diatas ke

![kucai](/assets/images/ci-cd/Screenshot 2025-12-09 224326.png)

jadi seperti contoh diatas ini masukan credentials yang sebelumnya sudah di buat jangan lupa atur branchnya ke main
setelah itu save,**NOTE** catat url yang ada di menu "Build when is pushed to gitlab" di bagian itu ada path penting yang mengarah ke project

```
http://192.168.56.169:8080/project/test2
```

bagian **/project/test2/** itu bagian penting catat dan simpan,jika sudah maka save
langkah selanjutnya ialah pergi ke gitlab lagi dan masuk ke repo yang tadi sudah di buat terus klik bagian 
settings lalu ke webhook lalu add new webhook

![logoo0812](/assets/images/ci-cd/result/Screenshot 2025-12-11 013403.png)

setelah itu akan ada tampilan seperti ini isi kolom url dengan url yang ngrok kasih sebelumnya lalu tambahkan sedikit diujung path dari project yang tadi di simpan dari config jobs di jenkins,lalu gabungkan,lalu kolom secret token didapat dari generate secret token paste ke url secret token setelah itu scroll kebawah

![logoii](/assets/images/ci-cd/result/Screenshot 2025-12-11 013418.png)

checklist semua yang ada karena ini hanya untuk test deployment dan buka production real jadi ini hanya experiment,lalu uncheck di bagian ssl biar lebih memudahkan,jika sudah maka save,setelah itu masu kembali ke bash lalu coba masukan file jenkins nya ke mode stagging,

```
git add .
git commit -m "commit"
git push origin main
```

seperti ini 

![logooo](/assets/images/ci-cd/result/Screenshot 2025-12-11 003826.png)

jika sudah bisa langsung di cek di bagian job di pipeline test2 maka akan ada job yang muncul karena berhasil triger lewat webhooknya
berikut beberapa tangkapan layar 

![logooo1](/assets/images/ci-cd/result/Screenshot 2025-12-11 003934.png)

![aoskd](/assets/images/ci-cd/result/Screenshot 2025-12-11 003949.png)

dan

![adi](/assets/images/ci-cd/result/Screenshot 2025-12-11 004010.png)

lalu

![iasabf](/assets/images/ci-cd/result/Screenshot 2025-12-11 004037.png)

dan hasil akhir jenkins file terdeploy

![tai](/assets/images/ci-cd/result/Screenshot 2025-12-11 012645.png)

dan ini hasil dari push yang tadi saya lakukan terbukti jenkins file berhasil di push lalu gitlab menerima webhook dan triger jenkinsfile 
setelah itu akan ada job yang jalan.Jadi selesai sudah saya menjelaskan workflow dari sistem deployment yang saya buat saya sistem ini merupakan 
pembelajarn dan experimen bagi saya yang penasaran tentang Ci/Cd workflow untuk kedepannya saya akan membuat yang lainya entah itu nantinya akan 
ada monitorng log atau menggunakan argo,tapi saya akan mendeploy nginx di postingan saya selanjutnya tunggu saja saya akan selalu membagikan 
tips dan materi pembelajaran saya senang bisa berbagi,kedepanya juga saya ingin sekali untuk menggunkan bare metal ketimbang vm saja saya 
meiliki laptop usang yang tidak terpakai saya memiliki niat untuk mengubah laptop itu menjadi home server nantinya,tunggu saja postingan selanjutnya dari saya tentang deployment menggunakan sistem ini,oh ya karena saya bukan dev jadi saya hanya deploy nginx karena simpel dan cepat 
saya juga masih belajar mungkin kedepanya saya akan coba deploy php saya akan melakukan beberapa variasi deployment agar tidak bosan heheh 😆😆😆
ohh ya saya menggunakan beberapa referensi jika kalian butuh 

- [JENKINS](https://www.jenkins.io/doc/book/installing/linux/)
- [NGROK](https://ngrok.com/download/linux)
- [KUBERNETES](https://kubernetes.io/docs/tasks/tools/)

TERIMAKASIH SUDAH MENYIMAK DAN MEMBACA SAMPAI AKHIR SEMOGA HARIMU MENYENANGKAN
