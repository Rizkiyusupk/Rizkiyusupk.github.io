---
title: "Debugging error Build And Testing Ci/Cd Hybrid in Rhel (Bare Metal + Vm ) "
date: 2026-04-06
tags: [kubernetes, linux, Ci/Cd, devops, containerd,ci,cd,ci/cd,system,infrastructure,Redhat,rhel]
header:
  teaser: /assets/images/redhat-ci-cd/Untitled design (5).jpg
categories: [DevOps, Kubernetes,linux,Linux,Server,Deploy,Deployment]
---

![asdasd](/assets/images/redhat-ci-cd/Untitled design (5).jpg)


### Overview 

Untuk post error kali ini saya akan secara khusus membahas soal error yang saya temukan ketika saya membuat projek [ini](https://rizkiyusupk.github.io/devops/kubernetes/linux/server/deploy/deployment/build-ci-cd-on-rhel/)
untuk error sendiri sebenarnya tidak terlalu jauh dengan projek membangun cluster di rhel,jadi langsung masuk saja 

### Interface tidak memiliki walaupun sudah di up 

nah ada kasus dimana ketika saya baru membuat sebuah vm untuk salah satu worker di cluster ini jadi momen ini terjadi ketika saya
baru saja menstart vm lalu ketika cek interface dari bridge adapter tidak ada ip sama sekali ketika di cobe untuk di up menggunakan 
command 

```
sudo nmcli connection up interface-name
|
sudo nmcli connectiion up ens256
```

output dari command yang seharusnya succesfull malah keluar

```
Error: unknown connection 'ens256'
```

nah itu terjadi karena interface belum di tambahkan ke dalam connection dari vm 

### Solve

jika masalah sudah di temukan maka saatny menyelesaikan masalah ini karena interface yang di maksud tidak dan belum di tambahkan jadinya
tambahkan saja gunakan command

```
sudo nmcli connection add type ethernet ifname interface-name con-name connection-name
|
sudo nmcli connection add type ethernet ifname ens256 con-name ens256
```

nah jika sudah coba gunakan kembali command

```
sudo nmcli connection up interface-name
|
sudo nmcli connection up ens256
```

nah lalu jika sudah cek ip dari si interface sekarang pasti sudah ada 

### Error Devices Allow 

Untuk eror selanjutnya yaitu eror device allow ini berhubungan dengan konfigurasi dari cgroup dan kernel yang ada di virtualbox
dikarenakan di postingan yang [ini](https://rizkiyusupk.github.io/devops/kubernetes/linux/server/deploy/deployment/build-kubernetes-in-redhat/) 
nah dan juga untuk container runtime interface menggunakan crio yang dimana environment dari virtualbox tidak mendukung adanya crio,mulai dari
cgroup rules ataupun kernel level configuration jadinya crio tidak bisa berjalan dengan baik di virtual box maka dari itu tools yang ada di gunakan pun berbeda dengan postingan yang lain 

### Solve

Untuk menyelesaikan masalah ini cukup sederhana tinggal ganti dari virtual box ke vmware bisa langsung cari saja vmware,atau [klik link ini](https://www.techspot.com/downloads/189-vmware-workstation-for-windows.html#google_vignette)
sudah selesai deh 

### **Jenkins file tidak tereksekusi di pipeline**

nah untuk selanjutnya eror yang terjadi ini bisa terjadi ketika pipeline sudah berjalan namun jenkins file tidak tereksekusi jadinya 
deployment berhenti, contohnya seperti ini

![lasda](/assets/images/errors/Screenshot 2026-03-03 144328.png)

nah eror di atas bisa saja terjadi ketika user jenkins tidak memiliki permission untuk mengeksekusi jenkinsfile,maksudnya kube config yang di copy dari node master lalu di paste di user jenkins itu tidak memiliki permission yang sesuai dengan kebutuhan user jenkins,
atau bisa juga dengan tidak adanya kube config di user jenkins nantinya node jenkins kebingungan bagaimana caranya mengekeskusi jenkins file
tanpa adanya kube config

### Solving problem

Sekarang masuk kebagian penyelesaian masalah untuk step pertama pastikan di dalam direktori user jenkins itu ada kube config

```

rizky@rizki:~$ ls -al  /var/lib/jenkins/
total 144
drwxr-xr-x 18 jenkins jenkins  4096 Mar  8 21:12 .
drwxr-xr-x 97 root    root     4096 Feb 26 21:17 ..
drwxr-xr-x  3 jenkins jenkins  4096 Jan  3 18:56 .cache
drwxr-xr-x  7 jenkins jenkins  4096 Jan 24 00:05 caches
-rw-r--r--  1 jenkins jenkins   789 Mar  8 21:12 com.dabsquared.gitlabjenkins.connection.GitLabConnectionConfig.xml
-rw-r--r--  1 jenkins jenkins   366 Jan  4 00:49 com.dabsquared.gitlabjenkins.GitLabPushTrigger.xml
drwxr-xr-x  3 jenkins jenkins  4096 Jan 17 19:07 .config
-rw-r--r--  1 jenkins jenkins  1660 Mar  8 21:12 config.xml
-rw-r--r--  1 jenkins jenkins  1975 Jan 17 17:12 credentials.xml
drwxr-xr-x  3 jenkins jenkins  4096 Jan 17 19:07 fingerprints
drwxr-xr-x  3 jenkins jenkins  4096 Jan  4 00:13 .groovy
-rw-r--r--  1 jenkins jenkins   156 Mar  8 21:12 hudson.model.UpdateCenter.xml
-rw-r--r--  1 jenkins jenkins   370 Jan  4 00:13 hudson.plugins.git.GitTool.xml
-rw-------  1 jenkins jenkins  1680 Jan  4 00:13 identity.key.enc
drwxr-xr-x  3 jenkins jenkins  4096 Jan  3 18:56 .java
-rw-r--r--  1 jenkins jenkins     7 Mar  8 21:12 jenkins.install.InstallUtil.lastExecVersion
-rw-r--r--  1 jenkins jenkins     7 Jan  4 00:24 jenkins.install.UpgradeWizard.state
-rw-r--r--  1 jenkins jenkins   179 Jan  4 00:24 jenkins.model.JenkinsLocationConfiguration.xml
-rw-r--r--  1 jenkins jenkins   171 Jan  3 18:57 jenkins.telemetry.Correlator.xml
drwxr-xr-x  6 jenkins jenkins  4096 Mar  2 21:41 jobs
drwxr-x---  3 jenkins jenkins  4096 Mar  3 15:30 .kube
-rw-r--r--  1 jenkins jenkins     0 Mar  8 21:12 .lastStarted
drwxr-xr-x  3 jenkins jenkins  4096 Jan 17 19:54 logs
-rw-r--r--  1 jenkins jenkins  1037 Mar  8 21:12 nodeMonitors.xml
-rw-r--r--  1 jenkins jenkins    46 Mar  3 22:01 org.jenkinsci.plugins.workflow.flow.FlowExecutionList.xml
-rw-r--r--  1 jenkins jenkins     4 Mar  3 22:32 .owner
drwxr-xr-x 95 jenkins jenkins 12288 Jan  4 00:49 plugins
-rw-r--r--  1 jenkins jenkins   259 Mar  3 23:56 queue.xml.bak
-rw-r--r--  1 jenkins jenkins    64 Jan  3 18:56 secret.key
-rw-r--r--  1 jenkins jenkins     0 Jan  3 18:56 secret.key.not-so-secret
drwx------  2 jenkins jenkins  4096 Jan 17 19:11 secrets
drwx------  2 jenkins jenkins  4096 Jan 17 17:01 .ssh
drwxr-xr-x  2 jenkins jenkins  4096 Mar  8 21:13 updates
drwxr-xr-x  2 jenkins jenkins  4096 Jan  3 18:57 userContent
drwxr-xr-x  3 jenkins jenkins  4096 Jan  4 00:24 users
drwxr-xr-x  6 jenkins jenkins  4096 Mar  2 21:56 workspace

```

nah jika ada .kube dalam direktori user jenkins maka ini hanya persoalan permission tapi jika belum ada .kube maka 
tinggal copy dari node master 

```
cat .kube/config
```

lalu copy outputdari config kube itu dan pastekan ke direktori jenkins,setelah itu set permission 
ke jenkins semua ya,
