---
title: "Build And Testing Ci/Cd Hybrid in Rhel (Bare Metal + Vm )"
date: 2026-04-03
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
| **Master**  | 5 cores | 8GB  | 25GB    | 2 Adapters (NAT + Bridged-Adapter)    |
| **Worker 1**| 5 cores | 6GB  | 25GB    | 2 Adapters (NAT + Bridged-Adapter)    |
| **Worker 2**| 5 cores | 6GB  | 25GB    | 2 Adapters (NAT + Bridged-Adapter)    |
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
jika token expire kamu bisa buat lagi dengan cara

```
sudo kubead create token create --prin-join-command
```

nah gunakan perintah diatas supaya token baru di buat

## SETUP BARE METAL

Dikarenakan ini menggunakan bare metal dan bukan vm disini tidak ada setup vm seperti biasanya,jadi saya menggunakan laptop lama saya yang 
sudah jarang di gunakan sebagai single node dan kebetulan os dari laptop lama saya itu ubuntu jadinya saya bisa memanfaatkan laptop usang saya 
sebagai bahan belajar
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

## **Jenkins Setup**

Pada tahap ini saya akan membuat kembali 1 vm baru untuk si jenkinsnya,sebenarnya bisa saja saya membuat versi containernya yang bisa di scale dikarenakan saya itu masih belajar untuk versi containernya akan saya tunda terlebih dahulu,untuk stepnya sama seperti membuat vm kube sebelumnya jadi bisa langsung skip ke installasi jenkinsnya
pertama-tama update lalu buat install openjdk jika java belum terinstall,kenapa butuh java karena jenkins tidak akan berjalan tanpa java 
[selengkapnya](https://www.jenkins.io/solutions/java/#:~:text=Jenkins%20supports%20building%20Java%20projects,the%20tool%20many%20years%20ago.)
,kita skip langsung ke instalasinya update package terlebih dahulu


```
sudo apt-get update
```

jika sudah bisa langsung install openjdknya 


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


```
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

check status jenkinsnya dengan 


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
ssh host@ip-port
misal
ssh rizki@192.168.56.167
```

jika sudah masuk ke terminal master ketik command

```
cat ~/.kube/config
```

output seharusnya seperti ini 

![logo7](/assets/images/redhat-ci-cd/Screenshot 2026-04-04 211409.png)

copy semua hasil dari output itu lalu ssh ke terminal jenkins dengan cara yang sama seperti ke kube master
jika sudah buat sebuah direktori untuk meyimpan file user jenkins


```
sudo mkdir -p ~/var/lib/jenkins/.kube/
```

jika sudah copy output tadi di masukan ke file di dalam direktori yang baru saja di buat


```
sudo vim ~/var/lib/jenkins/.kube/config
```

setelah itu ubah perizinan dari si filenya 


```
sudo chown -R jenkins:jenkins ~/var/lib/jenkins/.kube
```

atau gunakan saja scp untuk melakukan transfer keseluruhan direktori gunakan perintah

```
scp -r ./kube/ username@ip-node:/full/path/
|
scp -r ./kube/ rizky@192.168.100.7:/home/rizky/var/lib/jenkins/
```

lalu set ownershipnya seperti cara sebelumnya agar aman, 
jika sudah di paste test untuk uji coba akses ke kube gunakan command 

```
sudo -u jenkins kubectl get nodes
```

untuk test saja gunakan command diatas,output yang akan di keluar itu seperti ini

![logo3](/assets/images/redhat-ci-cd/Screenshot 2026-04-04 211542.png)

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
ip-node:port
misal
192.168.100.55:9091
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
ngrok http 9091
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
