---
title: "Debugging End-to-End Infrastructure Automation Designing Cross Platform DevOps Lab: K8s on QEMU/KVM with Terraform and Ansible"
date: 2026-06-22
tags: [kubernetes, linux, devops,containerd,system,infrastructure,Ci/Cd,KVM,QEMU,Terraform,terraform,Ansible]
header:
  teaser: /assets/images/kvm/INFRASTUCTURE.png
categories: [DevOps, Kubernetes,linux,Linux,Server,iac,infrastructure]
---

![akshergd](/assets/images/kvm/INFRASTUCTURE.png)

### Overview 

Masuk ke pembahasan error jadi dipost kali ini saya akan membahas error khusus yang saya temui ketika membuat projek KVM/QEMU ini langsung saja masuk ke pembahasanyaa

### SSH Host Unreachable

Karena network layer dan juga firewall yang ada didalam projek ini cukup kompleks jadinya ssh dari laptop 2 ke cluster yang berada didalam laptop 1 itu sulit dan sangat susah untuk dilakukan
dan juga ansible dalam projek ini itu berada didalam laptop 2 dan ansible bekerja menggunakan mekanisme ssh dan tanpe agent jadinya ansible sangat sangat mengandalkan koneksi ssh agar bisa
berjalan oleh karena itu saya sudah menemukan solusi dari permsalahan ini

### Solve

Dengan menggunakan mekanisme proxy jump,misalkan saya ssh ke laptop 1 dan laptop 1 meneruskan ke cluster yang ada du wsl jadinya ssh bisa bekerja dan ansible bisa berjalan untuk proxy jumpnya
sudah ada didalam docs projek ini jadinya tinggal mengikuti step by step dari proses pembuatan di docs saja atau jika ingin langsung bisa copy paste saja dari sini

```
vim ~/.ssh/config
|
Host hype-r
    HostName 192.168.50.106
    User rizki
    IdentityFile ~/.ssh/id_rsa

Host k8s-master
    HostName 10.10.10.55
    User ubuntu
    ProxyJump hype-r

Host k8s-worker1
    HostName 10.10.10.149
    User ubuntu
    ProxyJump hype-r

Host k8s-worker2
    HostName 10.10.10.161
    User ubuntu
    ProxyJump hype-r
```

### No Ssh Key Loaded

Oke kali ini masih seputar ssh sekarang ketika ingin mengeksekusi file terraform apapun akan keluar error sepert ini 

```
│ Error: Unable to Connect to Libvirt
│ 
│   with provider["registry.terraform.io/dmacvicar/libvirt"],
│   on main.tf line 8, in provider "libvirt":
│    8: provider "libvirt" {
│ 
│ An error occurred while connecting to libvirt.
│ 
│ URI: qemu+ssh://rizki@192.168.100.36/system
│ Error: failed to dial libvirt: ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain
│ no ssh password set
```

nah itu tandanya ssh key belum di load atau belum membuat eval.Karena Terraform pakai library SSH internal (bukan binary ssh), jadi dia tidak baca ~/.ssh/config dan tidak otomatis load 
key dari file.

```
Terraform → libvirt provider → koneksi ke qemu+ssh://rizki@192.168.100.36
                                        ↑
                        butuh SSH key yang sudah di-load di ssh-agent
```

Kalau ssh-agent tidak jalan dan key belum di-load, Terraform tidak tahu harus pakai key yang mana → error unable to authenticate.

### Solve 
Cukup buat sebuah eval atau ssh agent gunakan command

```
eval $(ssh-agent -s)
ssh-add .ssh/id_rsa
```

lalu enter nantinya akan menload ssh secara otomatis
