---
title: "Build And Testing Ci/Cd Hybrid On Kubernetes (Bare Metal + Vm )"
date: 2026-04/03
tags: [kubernetes, linux, Ci/Cd, devops, containerd,ci,cd,ci/cd,system,infrastructure,Redhat,rhel]
header:
  teaser: /assets/images/redhat-ci-cd/Untitled design (5).jpg
categories: [DevOps, Kubernetes,linux,Linux,Server,Deploy,Deployment]
---

![asd](/assets/images/redhat1/Untitled design (4).jpg)

### Overview 

Postingan kali ini akan membahas step by step membuat sebuah sistem Ci/Cd di rhel dengan kubernetes sebagai container orchestration,
pada postingan sebelumnya membahas installasi cluster kubernetes di rhel nah postingan kali ini akan membahas kelanjutannya,yang 
sebelumnya juga saya pernah membahas soal membuat Ci/Cd sistem di ubuntu karena saya penasaran jadinya saya ingin melanjutkan eksperimen saya
langsung saja ke pembahasannya

### Tools
- **Rhel (plow)** : 9.6
- **Kubernetes** : v1.30
- **Container Runtime Interface** : crio 1.30.10
- **CNI Plugin** : Flannel
- **Vmware Workstation** : 25h2
- **Docker** : 28.2.2
- **Rhel Account**
- **Gitlab** : Web
- **Git Bash** : git version 2.51.1.windows.1
- **Java Open-jdk**: 17.0.17
- **OS**: Ubuntu 25.04 (Bare Metal / Jenkins )
- **Jenkins**: 2.528.2

| Node        | CPU     | RAM  | Storage | Network                               |
|-------------|---------|------|---------|---------------------------------------|
| **Master**  | 5 cores | 8GB  | 25GB    | 2 Adapters (NAT + Host-Only-Adapter)  |
| **Worker 1**| 5 cores | 6GB  | 25GB    | 2 Adapters (NAT + Host-Only-Adapter)  |
| **Worker 2**| 5 cores | 6GB  | 25GB    | 2 Adapters (NAT + Host-Only-Adapter)  |
| **Jenkins** | 7 cores | 7GB  | 240GB   |                Wlan                   |


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
**NOTE!!! Ulangi setiap step diawal sampai ada 3 mesin virtual**

### Install Rhel

Masuk ke tahap selanjutnya yaitu bagian installasi rhel , di step ini khusus membahas soal step by step installasi rhel
start salah satu mesin virtual

![aishd](/assets/images/redhat1/Screenshot 2026-03-28 144922.png)

pilih opsi pertama diantara 3 opsi jika sudah kamu akan diarahkan nantinya ke

![asd](/assets/images/redhat1/Screenshot 2026-03-28 145104.png)

sebuah menu seperti diatas lalu pilih bahasa,saya akan menggunakan bahasa inggris dan keyboardnya juga sama,
jika sudah klik done lalu akan masuk ke menu installation summary

![aisbd](/assets/images/redhat1/Screenshot 2026-03-28 145129.png)

nah ada banyak hal yang perlu di setup dan konfigurasi maka dari itu yang pertama ialah disk masuk ke menu disk 

![asihd](/assets/images/redhat1/Screenshot 2026-03-28 145149.png)

lalu klik dua kali pada disk yang sudah ada,itu disk yang sebelumnya di set di bagian setup ukuran disk waktu setup mesin virtual
pastikan ada tanda ✔️ di bagian disk nya itu menandakan bahwa kamu sudah memilih disk itu sebagai tempat untuk menyimpan semua file dan 
konfigurasi,klik done lalu masuk ke root setup di bagian ini kita akan mensetup root password

![asodih](/assets/images/redhat1/Screenshot 2026-03-28 145220.png)

root password ini harus di catat karena ini penting,klik done jika semua sudah selesai
lalu masuk ke bagian setup user,masukan nama lengkap kamu lalu nama dari user yang akan di pakai 

![asihda](/assets/images/redhat1/Screenshot 2026-03-28 145251.png)

jangan lupa juga dengan passwordnya,bisa kamu checklist juga bagian **Make this user administrator** jika kamu ingin full permission 
untuk user ini,klik done masuk ke bagian registry untuk akun rhel nah perlu di catat jika di rhel itu perlu akun untuk menyelesaikan semua
setup dan konfigurasi dalam installasi,satu hal yang akan menjadi poin penting disini jika belum punya akun rhel buat terlebih dahulu 
gratis ko [klik disini saja](https://sso.redhat.com/auth/realms/redhat-external/login-actions/registration?client_id=rhcom&tab_id=IgMP15R1JZ4&client_data=eyJydSI6Imh0dHBzOi8vd3d3LnJlZGhhdC5jb20vZW4vZW5nYWdlL3NvbHZlLWJ1c2luZXNzLWNoYWxsZW5nZXMtcmhlbC0xMC1lYm9vaz9zY19jaWQ9UkhDVEEwMjUwMDAwNDY5ODI0JmdjbHNyYz1hdy5kcyZnYWRfc291cmNlPTEmZ2FkX2NhbXBhaWduaWQ9MjMzODExNjgxMTkmZ2JyYWlkPTBBQUFBQURzYlZNUzJRM256dDU5RFI4ZExURksyN0pzTlImZ2NsaWQ9Q2p3S0NBanctSjNPQmhCdUVpd0F3cVpfaDJsMDdyM2owdml5dXhWaWVDcVBrcjl3WUFWVlk3b3hWLVFCOWJqQ1d6T1BrWElrbjJtYWl4b0NBc2NRQXZEX0J3RSIsInJ0IjoiY29kZSIsInN0IjoiOTM0YjRmNzA1NWI2NGIyNWE2NzllNzM2NTU0ZTQ3NTgifQ)
jika sudah buat akun tinggal balik lagi kesini lalu masuk untuk registry akun supaya semua setup selesai 

![asdasd](/assets/images/redhat1/Screenshot 2026-03-28 145326.png)

klik register

![asdj](/assets/images/redhat1/Screenshot 2026-03-28 145558.png)


nah diatas saya sudah berhasil untuk registry akunnya sekarang ke tahap selanjutnya 
klik done lalu masuk ke menu network pastikan semua interface sudah on atau live agar nantinya mesin virtual bisa terhubung dengan internet ataupun
bisa berkomunikasi dengan host device dengan menggunakan NAT dan Host-Only-adapter

![asdb](/assets/images/redhat1/Screenshot 2026-03-28 145634.png)

nah sudah selesai untuk bagian jaringan atau networknya,klik done dan masuk ke tahap selanjutnya
yaitu bagian software yang akan ada di mesin virtual **NOTE!!semua mesin akan sama master maupun worker akan memilki software package yang sama**
```
pilih bagian server
|
- Basic Web Server
- Guest Agents
- DNS Name Server
- FTP Server
- Debugging Tools
- Virtualization Hypervisor
- Hardware Monitoring Utilities
- Windows File Server
- Infiniband Support
- Network Servers
- Mail Server
- File and Storage Server
- Performance Tools
- Network File System Client
- Remote Management for Linux
- Container Management
- RPM Development Tools
- Console Internet Tools
- .NET Development
- Legacy UNIX Compatibility
- Scientific Support
- Security Tools
- Development Tools
- System Tools
- Headless Management
```

diatas merupakan semua software yang perlu di install itu

![asdas](/assets/images/redhat1/Screenshot 2026-03-28 145911.png)

jika sudah klik done lagi lalu begin installation

![asdfd](/assets/images/redhat1/Screenshot 2026-03-28 150011.png)

tunggu sampai installasi selesai , jika sudah lakukan ke mesin yang lainnnya 

### Setup Kubernetes

Di tahap ini full akan membahas bagaimana proses step by step installasi kubernetes di rhel , 
karena rhel sudah terinstall seperti yang sudah di lakukan di step sebelumnya kali ini sampai di tahap installasi 
nah jika sudah terinstall langsung saja masuk dengan user yang di buat tadi ,jika sudah masuk ke node pertama yang akan di jadikan master
pastikan node master adalah node yang memliki 8 gb ram,
pertama-tama ganti dulu hostnamenya jadi master

```
sudo hostnamectl set-hostname master
```

gunakan command diatas untuk mengganti username dari yang tadinya localhost jadi master, jika sudah masuk ke swap karena kita akan menginstall
kubernetes full kita akan butuh yang namaya kubelet jadinya swap harus mati karena kubelet tidak akan berjalan jika ada swap,gunakan command

```
sudo swapoff -a
```

periksa dengan menggunakan command

```
free -h
```

atau jika mau permanen coba gunakan 

```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

atau kamu bisa masuk ke direktori dari swap itu di

```
sudo vim /etc/fstab
|
cari baris yang ada kata swap lalu diawal baris itu pake # untuk merubahnya jadi sebuah command
```

nah jika sudah masuk tahap selanjutnya yaitu dns,supaya semua mesin saling mengenal satu sama lain kita perlu mengkonfigurasi dns dari semua mesin
jadinya nanti semua mesin bisa saling kenal,langsung saja ke setiap mesin masuk ke direktori dari dns yang ada di 

```
sudo vim /etc/hosts
```

nah di dalam file itu masukan semua ip dan hostname yang akan di resolve,jadi semua ip mesin akan bisa di ketahui hanya dengan hostnamenya saja
jadinya kamu ga usah pake ip lagi tinggal pake hostname

```
contoh
|
192.168.1.1 master
192.168.1.2 worker
192.168.1.3 worker1
```

diatas merupakan contoh saja jadinya harus di sesuaikan dengan ip yang ada di interface host-only nah semuanya itu harus menggunakan interface host-only adapter tidak bisa menggunakan interface yang lain 
jadinya nantinya semua mesin bisa saling mengenal satu sama lain
lakukan seperti contoh diatas di setiap mesin , harus sesuai dengan interface host-only adapter
selanjutnya masuk ke bagian ssh, ssh berguna ketika nantinya kita perlu melakukan remote dari satu mesin ke mesin lainnya jadinya ini juga berguna
memang tidak wajib tapi lebih baik ada , gunakan command

```
ssh-keygen -t rsa
```

gunakan command diatas untuk membuat ssh key,biarkan semuanya default jadi tekan enter sampai semuanya selesai lalu jika sudah copy ke masing-masing mesin gunakan command

```
ssh-copy-id -i /path/to/file username@hostname
|
ssh-copy-id -i /home/iki/.ssh/id_rsa.pub iki@worker
```

nah aslinya harus menggunakan ip di ujung karena sebelumnya sudah membuat dns resolve untuk mempermudah dalam resolve ip jadi hostname tinggal masukan saja hostnamenya apa karena memang tidak butuh ip lagi
lakukan step ini dengan urutan 

```
master  -> worker
master  -> worker1
worker  -> master
worker  -> worker1
worker1 -> master
worker1 -> worker
```

jadinya semua mesin punya masing masing ssh key dari keseluruhan cluster,
next untuk tahap selanjutnya yaitu port nah karena ini kubernetes jadinya butuh beberapa port yang khusus 
buat master

```
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=2379-2380/tcp
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=10251/tcp
sudo firewall-cmd --permanent --add-port=10252/tcp
sudo firewall-cmd --reload
```

lalu di semua worker

```
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=30000-32767/tcp                                                 
sudo firewall-cmd --reload
```

jika sudah maka akan masuk ke tahap selanjutnya yaitu mengubah selinux dari enforcing jadi permissive
masuk ke 

```
sudo vim /etc/selinux
|
SELINUX=enforcing -> permissive
```

nah jika sudah sekarang tinggal masuke tahap selanjutnya masuk ke tahap modules 
dimana kita akan membuat modules yang diperlukan untuk kubernetes full penjelasan mengenai module nya ada postingan [ini](https://rizkiyusupk.github.io/devops/kubernetes/linux/server/hybird-ci-cd/) 
nah langsung saja masuk ke direktorinya 

```
sudo vim /etc/modules-losd.d/k8s.conf
|
overlay
br_netfilter
```

jika sudah gunakan command modprobe untuk menload modules tadi

```
sudo modprobe overlay
sudo modprobe br_netfilter
```

jika sudah maka masuk ke tahap selanjutnya yaitu sysctl parameters  masuk ke direktori 

```
sudo vim /etc/sysctl.d/k8s.conf
|
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
```

jika sudah gunakan command 

```
sudo sysctl --system
```

untuk menerapkan parameters yang sudah di set,
langsung saja masuk ke tahap selanjutnya yaitu installasi cri(Container Runtime Interface ) jika di postingan sebelumnya sering sekali menggunakan
containerd kali ini saya akan menggunakan cri-o sebagai cri karena saya menggunakan distro rhel jadinya lebih enak menggunakan crio yang lebih native
ketimbang menggunakan containerd,
langsung saja pertama-tama buat sebuah global variable

```
export CRIO_VERSION=v1.30
```

setelah itu masukan repository dari cri-o 

```
cat <<EOF | sudo tee /etc/yum.repos.d/cri-o.repo
[cri-o]
name=CRI-O
baseurl=https://pkgs.k8s.io/addons:/cri-o:/stable:/$CRIO_VERSION/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/addons:/cri-o:/stable:/$CRIO_VERSION/rpm/repodata/repomd.xml.key
EOF
```

jika sudah langsung saja install dengan command

```
sudo dnf install cri-o -y 
```

tunggu hingga installasinya selesai jika sudah kamu bisa lansung ketik command ini

```
sudo systemctl enable crio
sudo systemctl start crio
```

command diatas berfungsi untuk mengenable dan menstart program crio agar bisa langsung running
jika sudah sekarang masuk ke tahap selanjutnya yaitu installasi kubernetes , pertama-tama buat global environmentnya terlebih dahulu 

```
KUBERNETES_VERSION=v1.30
```

jika sudah tambahkan repository kubernetesnya 

```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/rpm/repodata/repomd.xml.key
EOF
```

jika sudah langsung saja install 3 komponen utama dari kubernetes yaitu kubeadm , kubelet , kubectl

```
sudo dnf install kubelet kubeadm kubectl -y
```

tunggu hingga installasi selesai **NOTE LAKUKAN SEMUA TAHAP DI ATAS KE SEMUA MESIN TANPA TERKECUALI**
### Init
**NOTE LAKUKAN TAHAP INI CUMAN DI MASTER NODE SAJA**
tahap selanjutnya ialah init kubernetes untuk membuat token
gunakan command ini

```
sudo kubeadm init \
  --apiserver-advertise-address=192.168.xxx.xxx \
  --pod-network-cidr=10.244.0.0/16 \
  --node-name=nama-vm-mastermu
```

jika sudah akan ada instruksi untuk membuat sebuah direktori untuk kubernetes

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

ikuti perintah diatas untuk membuat sebuah home direktori untuk kubernetes,
nah akan ada token yang bisa di copy paste lalu di jalankan di node worker untuk join ke cluster
jika sudah install flannel untuk network  

```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

tunggu sampai tahap installasi selesai maka kubernetes sudah terinstall di RHEL
