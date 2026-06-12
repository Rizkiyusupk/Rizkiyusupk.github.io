---
title: "Debugging IaC: The Final Phase Achieving Full Automation Setup Ci/Cd System"
date: 2026-06-12
tags: [kubernetes, linux, devops, ansible,iac,system,infrastructure,]
header:
  teaser: /assets/images/iac2/Untitled design (6).jpg
categories: [DevOps, Kubernetes,linux,Linux,Server,iac,infrastructure]
---

![akshd](/assets/images/iac3/INFRASTUCTURE.png)

### Overview

Sampai juga dibagian paling seru yaitu debuging error yang dihadapi ketika membuat projek [ini](https://rizkiyusupk.github.io/devops/kubernetes/linux/server/iac/infrastructure/iac3/)
oke tanpa berlama-lama langsung saja masuk ke pembahasanya

### Inkonsistensi tempat pemyimpanan gpg key

Oke dibeberapa projek yang berkaitan dengan jenkins ada step atau langkah dimana harus menambahkan gpg key dari jenkins terlebih dahulu sebelum melakukan installasi jenkins itu sendiri
untuk bagian spesifiknya itu 

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

pada saat melakukan curl directory yang diarahkan itu ke 

```
 /usr/share/keyrings/jenkins-keyring.asc
```

tetapi sistem mengecek ke directory 

```
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian binary/" | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
```

sistem mengecek ke

```
/etc/apt/keyrings/jenkins-keyring.asc
```

maka akan terjadi error ketika mengeksekusi file playbook untuk installasi jenkins untuk detail error message 

```
fatal: [node2]: FAILED! => {"changed": true, "cmd": ["apt", "update"], "delta": "0:00:33.787515", "end": "2026-06-03 03:00:17.249608", "msg": "non-zero return code", "rc": 100, "start":
"2026-06-03 02:59:43.462093", "stderr": "\nWARNING: apt does not have a stable CLI interface. Use with caution in scripts.\n\nW: GPG error: https://pkg.jenkins.io/debian binary/ Release:
The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 7198F4B714ABFC68\nE: The repository 'https://pkg.jenkins.io/debian binary/ Release' is not
signed.", "stderr_lines": ["", "WARNING: apt does not have a stable CLI interface. Use with caution in scripts.", "", "W: GPG error: https://pkg.jenkins.io/debian binary/ Release: The
following signatures couldn't be verified because the public key is not available: NO_PUBKEY 7198F4B714ABFC68", "E: The repository 'https://pkg.jenkins.io/debian binary/ Release' is not
signed."], "stdout": "Ign:1 https://pkg.jenkins.io/debian binary/ InRelease\nGet:2 https://pkg.jenkins.io/debian binary/ Release [2,044 B]\nGet:3 https://pkg.jenkins.io/debian binary/
Release.gpg [833 B]\nHit:4 http://id.archive.ubuntu.com/ubuntu noble InRelease\nHit:5 http://id.archive.ubuntu.com/ubuntu noble-updates InRelease\nHit:6
http://id.archive.ubuntu.com/ubuntu noble-backports InRelease\nHit:7 http://security.ubuntu.com/ubuntu noble-security InRelease\nIgn:3 https://pkg.jenkins.io/debian binary/
Release.gpg\nReading package lists...", "stdout_lines": ["Ign:1 https://pkg.jenkins.io/debian binary/ InRelease", "Get:2 https://pkg.jenkins.io/debian binary/ Release [2,044 B]", "Get:3
https://pkg.jenkins.io/debian binary/ Release.gpg [833 B]", "Hit:4 http://id.archive.ubuntu.com/ubuntu noble InRelease", "Hit:5 http://id.archive.ubuntu.com/ubuntu noble-updates
InRelease", "Hit:6 http://id.archive.ubuntu.com/ubuntu noble-backports InRelease", "Hit:7 http://security.ubuntu.com/ubuntu noble-security InRelease", "Ign:3 https://pkg.jenkins.io/debian
binary/ Release.gpg", "Reading package lists..."]}
```


### Solve

Jadi untuk memperbaiki inkonsistensi yang telah saya lakukan dan ini murni kecerobohan saya saya tidak memperhatikan kemana saya mengarahkan dan mencari gpg key dari jenkins maka
saya perbaiki semua projek yang berhubungan atau menggunakan tools jenkins khususnya dibagian install gpg key yang tadinya

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

lalu 

```
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian binary/" | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
```

menjadi

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key | sudo tee \
     /etc/apt/keyrings/jenkins-keyring.asc  > /dev/null
```

dan

```
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### Expired Gpg 

Masuk ke error selanjutnya,pada error kali ini yang menjadi penyebab dari error ini adalah gpg key yang expired coba gunakan command

```
gpg --show-keys /etc/apt/keyrings/jenkins-keyring.asc
```

lalu liat output dari command tersebut 

```
iki@node-2:~$ gpg --show-keys /etc/apt/keyrings/jenkins-keyring.asc gpg: directory '/home/iki/.gnupg' created gpg: keybox '/home/iki/.gnupg/pubring.kbx' created pub rsa4096 2023-03-27
[SC] [expired: 2026-03-26] 63667EE74BBA1F0A08A698725BA31D57EF5975CA uid Jenkins Project <jenkinsci-board@googlegroups.com> sub rsa4096 2023-03-27 [E] [expired: 2026-03-26] gpg: WARNING:
No valid encryption subkey left over.
```

jika output sama seperti diatas maka 100% penyebab dari error adalah tahun gpg key karena lihat pada bagian output terdapat kata expired 

### Solve

Oke karena sudah ditemukan akar masalah dari errornya sekarang bagian solving cukup ganti tahun key ke 2026 hehe,tenang jika menyimak dan membaca dengan seksama di docs projek 
semua sudah ditulis dan diperbaiki jadinya bukan masalah besar atau mau copy paste dari command diatas itu sudah cukup

### Start request repeated too quickly

Untuk error ini juga pada dasarnya sama yaitu mengenai versi,jadi error ini disebabkan oleh jenkins yang terus menerus resatart secara cepat dan berulang-ulang kenapa? karena versi dari 
java yang terlalu lama,saya baru tau mengenai hal ini perbedaan versi yang signifikan menyebabkan error sefatal ini,

```
Jun 03 06:09:33 node-2 jenkins[3805]: Running with Java 17 from /usr/lib/jvm/java-17-openjdk-amd64, which is older than the minimum required version (Java 21). Jun 03 06:09:33 node-2
jenkins[3805]: Supported Java versions are: [21, 25]
```

kurang lebih seperti diatas error messagenya jadi versi java terlalu lama 

### Solve

Karena sudah ditemukan akar masalahnya tinggal masuk ke bagian solvingnya ganti saja versi javanya dari yang 17 ke 21 (minimum),lihat baik baik di playbook dari projek
dibagian installasi jenkins java sudah menjadi 21,atau mau langsung bisa saja cukup gunakan command

```
sudo apt update
sudo apt install openjdk-21-jdk -y
```

### Auth as Anonymous 

Error selanjutnya masuk ke bagian jenkins,ketika ingin memeriksa apakah jenkins sudah berjalan atau tidak biasanya saya menggunakan command

```
curl localhost:8080
```

namun ketika error ini muncul malah outputnya seperti ini

```
Error from server (Forbidden): <html><head><meta http-equiv='refresh' content='1;url=/login?from=%2Fswagger-2.0.0.pb-v1%3Ftimeout%3D32s'/><script id='redirect' data-redirect-url='/login?
from=%2Fswagger-2.0.0.pb-v1%3Ftimeout%3D32s' src='/static/e16ea2cd/scripts/redirect.js'></script></head><body style='background-color:white; color:white;'> Authentication required <!--
You are authenticated as: anonymous Groups that you are in: anonymous Permission you need to have (but didn't): hudson.model.Hudson.Read ... which is implied by:
hudson.security.Permission.GenericRead ... which is implied by: hudson.model.Hudson.Administer -->
```

Karena sedikit aneh jadi saya kira itu merupakan error mengenai akses user apakahh user memiliki akses atau tidak,ternyata saya sedikit keliru error ini merupakan error jenkins tidak bisa
deploy ke cluster maupun trigger pipeline karena kube config tidak ada di directory jenkins

### Solve

Penyebeb error sudah ditemukan tinggal eksekusi , karena akar masalahnya sudah ditemukan yaitu kube config yang tidak ada di directory jenkins maka dari itu pindahkan kube config
dari control plane di cluster ke jenkins node bisa copy langsung output kube config atau gunakan scp


### Network CIDR fail

Masuk ke error selanjutnya error ini menyebabkan kubec flannel tidak mau untuk start dan malah erorr  **Back-off restarting failed**,jadinya container dan pods tidak akan memiliki network
karena flannel yang berperan sebagai network di cluster malah down dan error

```
Normal   Scheduled         9m15s                  default-scheduler  Successfully assigned kube-flannel/kube-flannel-ds-ndfsb to node1
  Normal   Pulled            9m13s                  kubelet            Container image "ghcr.io/flannel-io/flannel-cni-plugin:v1.9.1-flannel1" already present on machine
  Normal   Created           9m13s                  kubelet            Created container install-cni-plugin
  Normal   Started           9m11s                  kubelet            Started container install-cni-plugin
  Normal   Pulling           9m9s                   kubelet            Pulling image "ghcr.io/flannel-io/flannel:v0.28.5"
  Normal   Pulled            6m40s                  kubelet            Successfully pulled image "ghcr.io/flannel-io/flannel:v0.28.5" in 2m28.751s (2m28.752s including waiting)
  Normal   Created           6m40s                  kubelet            Created container install-cni
  Normal   Started           6m40s                  kubelet            Started container install-cni
  Normal   Pulled            6m35s (x2 over 6m38s)  kubelet            Container image "ghcr.io/flannel-io/flannel:v0.28.5" already present on machine
  Normal   Created           6m35s (x2 over 6m38s)  kubelet            Created container kube-flannel
  Normal   Started           6m34s (x2 over 6m37s)  kubelet            Started container kube-flannel
  Warning  BackOff           6m31s                  kubelet            Back-off restarting failed container kube-flannel in pod kube-flannel-ds-ndfsb_kube-flannel(cf5ac8f6-cf5f-4af9-93c6-
242dd41c6f48)
  Warning  DNSConfigForming  4m6s (x27 over 9m15s)  kubelet            Nameserver limits were exceeded, some nameservers have been omitted, the applied nameserver line is: 10.0.2.3
192.168.100.1 2001:4489:306:101::2
coba kamu liat ini
```

diatas merupakan full error logs,erorr terjadi karena format init salah dan tidak sesuai dengan format yang diharapkan oleh flannel sebagai network cluster biasanya init cuman seperti ini

```
kubeadm init --apiserver-advertise-address=192.168.2.35 --node-name=node1
```

init diatas yang menyebabkan error , saya kira bisa tanpa format cidr untuk flannel tetapi hasilnya malah error,karena projek ini memiliki konsep iac jadinya saya mengabaikan format cidr
flannel dan error itupun muncul :v 

### Solve

Oke tinggal tambahkan 

```
--pod-network-cidr=10.244.0.0/16 
```

pada init di playbook semua projek yang berkaitan dengan iac sudah di tambahkan format cidr flannel ini 
