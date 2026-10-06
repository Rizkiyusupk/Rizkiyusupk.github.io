---
title: "Debugging Error:Building a Pull-Based GitOps Pipeline: Jenkins CI + ArgoCD CD "
date: 2026-10-06
tags: [Clouds,linux, devops,system,infrastructure,Terraform,terraform,Argocd,k8s,kubernetes,Jenkins,Cloud,Infra]
header:
  teaser: /assets/images/pull-based/INFRASTUCTURE (3).jpg
categories: [DevOps, Clouds,linux,Linux,Server,iac,infrastructure]
---

![aihvsdv](/assets/images/pull-based/Neon DevOps Architecture Blueprint.png)

### Overview
Baik seperti biasa jika sebuah projek sudah selesai maka tahap selanjutnya yaitu pembuatan error pagenya tidak usa berlama-lama langsung saja


### Missing Kubeconfig

Error kali ini terjadi ketika ingin menginstall argocd,di langkah pertama dalam melakukan installasi dari argocd perlu namespace terlebih dahulu
nah untuk membuat namespace itu butuh command menggunakan kubectl,kubectl akan terhubung dengan api k8s yang nantinya perintah pembuatan dari 
namespace akan diteriman oleh etcd lalu namespace akan dibuat,nah untuk menggunakan kubectl dibutuhkan kubeconfig sebagai akses untuk bisa 
menggunakan kubectl,di kasus kali ini kubeconfig kan di simpan di direktori user tetapi di playbook sering menggunakan user root karena setiap 
installasi tidak membutuhkan permission dikarenakan root itu sendiri sudah berada di paling atas dan memiliki akses kemanapun jadinya ketika
ingin melakukan insallasi apapun bisa dilakukan tanpa khawatir adanya issue tentang permission,nah karena hal itu lah ada error ini
saya lupa message lenkapnya tapi intinya ansible mencari kubeconfig di direktori root 


### Solve
Untuk akar masalahnya sudah ada yaitu di user yang ada di playbook jadinya ya tinggal ganti saja user yang akan di pakai dengan user yang menyimpn
kubeconfig jika di saya itu akan menjadi seperti ini

```
become = true
|
beocome_user = ubuntu
```

untuk fullnya akan seperti ini

```
vim argocd.yaml
|
- name: install argocd
  hosts: masters,masters-cluster-2
  become: yes
  become_user: ubuntu

  tasks:
    - name: create namespace
      ansible.builtin.shell: kubectl create namespace argocd
    - name: install argo
      ansible.builtin.shell: kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### File Not Found
Ketika Jenkinsfile masuk ke stage dimana clone config-repo salah satu proses di stage clone config repo yaitu edit manifest dengan mengganti tag
image supaya ada perubahan yang ada di manifest lalu argocd mendeteksi adanya perubahan itu dan apply perubhan manifest ke cluster,nah disaaat 
setelah clone repo lalu akan mengedit file manifest jenkins akan kebingungan dan mengeluarkan error berupa file not found karena config repo 
masih kosong dan tidak ada apapun didalam nya maka dari itu diperlukan untuk melakukan commit manual terlebih dahulu agar jenkins tidak 
kebingungan


### Solve 
oke akar masalah sudah di temukan tinggal solve saja dan simple saja tinggal lakukan commit mau itu manual atau clone terlebih dahlu lalu commit

### Pods In Argocd Got Duplicate Automaticly
Saya mengalami hal ini ketika saya ingin masuk ke dashboard argocd,ketika ini saya terus terusan terpental dari dashboard bahkan tidak bisa masuk 
ke dashboard lalu saya curiga saya cek ke namespace argocd dan benar saja ada anomali di dalam namespace yaitu pods arocd terduplikat semua 
dan statusnya campuran ada yang running dan ada yang status unknown,saya curiga bahwa ini terjadi ketika saya mematikan cluster dengan cara close
tab dari wsl secara tiba tiba jadinya argocd tidak bisa memproses shutdown system,dan dugaan saya benar saya lakukan hal itu beberapa kali benar
saja pods terus terduplikat,pods terus terusan ditimpa dengan yang baru dan yang lama menjadi crash

![asicbsvn](/assets/images/pull-based/Screenshot 2026-10-02 205335.png)

bsia dilihat bahwa ada banyak pods di namespcae argocd dan statusnya campuran 

### Solve
Akar masalah sudah ditemukan tinggal eksekusi saja,gampang saja untuk menyelsaikan masalah ini tinggal hapus pods yang bermasalah ya karena 
memang pods yang memiliki status unknown itu ya yang bermasalah dan itu pods lama 
