---
title: "Automation Kubernetes Deployment Using Ansible (Full Infrastructure Setup)"
date: 2025-11-06
tags: [kubernetes, ansible, automation, devops, infrastructure, linux]
header:
  teaser: /assets/images/ansible/Untitled design.jpg
---
![kube](/assets/images/ansible/Untitled design.jpg)


# 🧩 Ringkasan Kubernetes dan Ansible

## Kubernetes
Kubernetes (K8s) adalah **platform orkestrasi container** yang digunakan untuk **mengelola, men-deploy, menskalakan, dan memantau aplikasi berbasis container** seperti Docker.

### Fungsi utama Kubernetes:
- **Deployment otomatis** – menjalankan container di node yang sesuai.
- **Auto-scaling** – menyesuaikan jumlah pod secara dinamis.
- **Load balancing & service discovery** – mendistribusikan trafik antar container.
- **Self-healing** – mengganti container yang gagal secara otomatis.
- **Rolling update** – memperbarui aplikasi tanpa downtime.

Dengan Kubernetes, pengelolaan aplikasi menjadi **lebih efisien, stabil, dan dapat diskalakan** secara dinamis.

---

## Ansible
Ansible adalah **alat otomatisasi konfigurasi dan orkestrasi sistem** yang bersifat **agentless** (tanpa agen tambahan pada host target).  
Tool ini digunakan untuk **mengelola konfigurasi server, deployment aplikasi, dan manajemen infrastruktur** secara otomatis dan konsisten.

### Ciri utama Ansible:
- Menggunakan **SSH** sebagai koneksi utama (tanpa agen di tiap server).
- Menggunakan **YAML (playbook)** untuk mendefinisikan task.
- Bersifat **idempotent**, menjalankan ulang playbook tidak mengubah konfigurasi yang sudah sesuai.
- Dapat mengatur ratusan server secara paralel dari satu kontrol pusat.

Dengan Ansible, administrator dapat menerapkan konsep **Infrastructure as Code (IaC)**, yaitu menjadikan konfigurasi sistem sebagai kode yang dapat dikelola dan diautomasi.

---

## Hubungan Kubernetes dan Ansible
Kubernetes mengelola **container dan aplikasi yang berjalan**, sementara Ansible mengelola **konfigurasi dan infrastruktur** tempat Kubernetes dijalankan.

### Contoh penerapan:
- Menyiapkan node untuk cluster Kubernetes.
- Mengonfigurasi jaringan cluster (CNI plugin).
- Mengotomatiskan deployment file manifest Kubernetes (.yaml).
- Integrasi dengan CI/CD pipeline.

Kombinasi Kubernetes dan Ansible menciptakan **proses deployment otomatis yang efisien, stabil, dan dapat diulang**.

---

## Mengapa Automation Kubernetes Menggunakan Ansible Itu Penting

1. **Mempermudah Deployment Cluster**  
   Proses instalasi Kubernetes manual cukup kompleks. Dengan Ansible, semuanya dapat dijalankan otomatis melalui satu playbook.

2. **Menjamin Konsistensi Konfigurasi**  
   Semua node dalam cluster mendapat konfigurasi yang sama, menghindari human error.

3. **Meningkatkan Skalabilitas dan Reusabilitas**  
   Menambah node baru atau membuat ulang cluster cukup menjalankan ulang playbook yang sama.

4. **Integrasi dengan CI/CD Pipeline**  
   Ansible bisa digunakan untuk memperbarui aplikasi otomatis ke cluster Kubernetes.

5. **Mendukung Infrastructure as Code (IaC)**  
   Semua konfigurasi disimpan dalam bentuk kode, mudah dikelola dengan Git dan kolaborasi tim.


nah berikut ini beberapa keuntungan dalam penggunaan automation ini

| No | Keuntungan | Penjelasan Singkat |
|----|-------------|--------------------|
| 1 | **Efisiensi Waktu Deployment** | Proses instalasi dan konfigurasi cluster dapat dilakukan otomatis melalui playbook Ansible, menghemat waktu setup yang biasanya manual dan panjang. |
| 2 | **Konsistensi Konfigurasi** | Semua node Kubernetes memiliki konfigurasi identik karena Ansible menerapkan prinsip *idempotent*, mencegah perbedaan setting antar server. |
| 3 | **Kemudahan Skalabilitas** | Penambahan node baru atau re-deploy cluster cukup menjalankan ulang playbook, tanpa perlu konfigurasi ulang dari awal. |
| 4 | **Integrasi CI/CD Lebih Mudah** | Ansible dapat dijalankan dalam pipeline CI/CD untuk menerapkan perubahan aplikasi otomatis ke cluster Kubernetes. |
| 5 | **Penerapan Infrastructure as Code (IaC)** | Konfigurasi sistem disimpan dalam bentuk kode YAML yang mudah dilacak, diuji, dan dikelola melalui Git. |
| 6 | **Reliabilitas dan Reproduksibilitas Tinggi** | Seluruh proses setup dan deployment dapat diulang dengan hasil yang sama di lingkungan berbeda (development, staging, production). |
| 7 | **Manajemen Cluster Terpusat** | Ansible memungkinkan pengelolaan banyak node dari satu kontrol pusat, mempermudah administrasi cluster berskala besar. |
| 8 | **Integrasi Multi-Platform** | Dapat diterapkan di berbagai sistem (Linux, cloud provider, atau bare metal) tanpa harus mengubah banyak konfigurasi. |

![kube](/assets/images/ansible/Untitled design.jpg)

---
## SETUP KUBERNETES
## WARNING DIKARENAKAN INI ADA K8S AKAN SANGAT BERAT UNTUK SISTEM ANDA DITAMBAH DISINI AKAN MEMAKAI ANSIBLE SEBAGAI AUTOMATIONNYA
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

jika sudah bisa kita langsung ke tahap selanjutnya,apply GPG key dan repository dari si kube

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

lalu setelah itu bisa langsung join

```
sudo kubeadm join 192.168.xxx.xxx:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

cari output itu diantara output **sudo kubedm init** tadi pastinya sudah sesuai dengan vm master mu,setelah itu copy lalu paste di vm worker 

## SETUP ANSIBLE
Jika sudah selesai dengan setup k8s atau kubernetesnya saatnya setup bagian ansible,dikarenakan si ansible ini menggunakan mekanisme ssh
atau agentless jadi disini wajib untuk mengenerate token ssh atau ssh-keygen

```
ssh-keygen
```

nah setelah kamu mengetik itu kamu akan di beri pilihan untuk enter passpharse nah disini kamu enter saja sampai prosesnya selesai
biarkan si key nya default

![logo12](/assets/images/ansible/Screenshot 2025-11-11 004332.png)

seperti contoh di atas itu kamu bisa lihat bahwa saya tidak memasukan apapun atau bisa dibilang biarkan saja default
jika sudah masuk ke command selanjutnya yaitu copy ssh keygennya ke node lainnya karena ini di perlukan 

```
ssh-copy-id -i /home/rizki/.ssh/id_ed25519.pub rizki@192.168.56.160
```

ganti full path file pubnya dengan yang kamu miliki dan ganti nama host serta ip nya dengan ip vm yang kamu miliki
jika sudah tekan enter,maka akan ada sebuah konfirmasi

![logo13](/assets/images/ansible/Screenshot 2025-11-11 005057.png)

nah dalam konfirmasi itu kamu bisa ketik yes lalu masukan password buat dari si nodenya
jika sudah lakukan lagi sampai ke semua node merata,bukan hanya di master saja tapi dari 
- master -> worker
- master -> worker1
- worker -> master
- worker -> worker1
- worker1 -> master
- worker1 -> worker

nah kurang lebih seperti itu,perintah perintah tersebut berguna untuk ssh agar tidak perlu adanya input password lagi dan 
dikarenakan cara kerja ansible ini dengan ssh maka di butuhkan mekanisme passwordless agar nantinya memudahkan proses dari si automationya 
untuk mengetesnya kamu cukup lakukan ssh ke nodenya

```
ssh rizki@192.168.56.161
```

nah jika si output seperti ini

![logo14](/assets/images/ansible/Screenshot 2025-11-11 013535.png)

nah jika sudah di konfirmasi bahwa sudah selesai dengan ssh sekarang masuk ke tahap setup dan konfigurasi ansible
cek apakah python sudah terinstall di os linux biasanya jika menginstall linux akan sepaket dengan python,tapi jika belum bisa install menggunakan apt

```
sudo apt-get update

sudo apt install python3 -y
```

nah kenapa python dibutuhkan di ansible ya karena ansible sendiri itu di buat atau di bangun diatas python jadi kita perlu python
untuk menggunakannya [selengkapnya](https://www.redhat.com/en/blog/ansible-coding-programming?utm_source=chatgpt.com),
nah untuk step selanjutnya kita perlu yang namanya venv untuk env ansiblenya nanti dan juga pip untuk install beberapa dependensi

```
sudo apt install python3-venv python3-pip -y
```

jika sudah install venv dan pip kamu bisa langsung buat direktori tempat venv di simpan misal

```
mkdir ansible-k8s
```

selanjutnya masuk ke direktori yang sudah dibuat

```
cd ansible-k8s
```

terus inisiasi untuk venv di direktori tersebut 

```
python3 -m venv ansible-venv
```

command diatas akan menginisiasi venv untuk env si ansible dan jika ingin langsung masuk ke venv bisa activate menggunakan command

```
source ansible-venv/bin/active
```

nah command diatas akan membuat terminal masuk ke venv yang sudah di inisiasi sebelumnya
dan masuk ke step berikutnya tinggal install beberapa dependensi untuk ansible core dan kubernetes menggunakan pip tentunya 
karena kita akan menginstallnya di dalam venv

```
## untuk upgrade si pipnya
pip install --upgrade pip

## dependesi si ansible
pip install ansible

## libraries buat kuber di python
pip install kubernetes openshift PyYAML

```

nah jika sudah kamu bisa langsung membuat inventorynya untuk mengetahui yang mana saja node yang akan di manage
menggunakan vim atau nano atau text editor manapun dengan format .ini

```
vim inventory.ini

[master]
master ansible_host=192.168.56.160 ansible_user=rizki
[workers]
worker ansible_host=192.168.56.161 ansible_user=rizki
worker1 ansible_host=192.168.56.163 ansible_user=rizki

```

ganti ke nama host mu dan ip node mu setelah itu test ping 

![logo16](/assets/images/ansible/Screenshot 2025-11-11 023605.png)
