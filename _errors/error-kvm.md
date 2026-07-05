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


### Error TLS 

Error kali ini disebabkan oleh tls yang issue karena tls issue jika menggunakan command

```
rizky@rizki:~$ sudo -u jenkins kubectl get nodes
E0630 19:20:20.357565    7383 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://127.0.0.1:6443/api?timeout=32s\": tls: failed to verify
certificate: x509: certificate is valid for 10.96.0.1, 10.10.10.55, not 127.0.0.1"
E0630 19:20:20.371186    7383 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://127.0.0.1:6443/api?timeout=32s\": tls: failed to verify
certificate: x509: certificate is valid for 10.96.0.1, 10.10.10.55, not 127.0.0.1"
E0630 19:20:20.395473    7383 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://127.0.0.1:6443/api?timeout=32s\": tls: failed to verify
certificate: x509: certificate is valid for 10.96.0.1, 10.10.10.55, not 127.0.0.1"
E0630 19:20:20.441837    7383 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://127.0.0.1:6443/api?timeout=32s\": tls: failed to verify
certificate: x509: certificate is valid for 10.96.0.1, 10.10.10.55, not 127.0.0.1"
E0630 19:20:20.460680    7383 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://127.0.0.1:6443/api?timeout=32s\": tls: failed to verify
certificate: x509: certificate is valid for 10.96.0.1, 10.10.10.55, not 127.0.0.1"
Unable to connect to the server: tls: failed to verify certificate: x509: certificate is valid for 10.96.0.1, 10.10.10.55, not 127.0.0.1
```

akan keluar ouput error seperti diatas,kenapa bisa issue karena scope tls nya tidak mencakup ip dari 127 jadinya secara default tls hanya mencakup ip dari node dan ip dari cluster,ketika
tls dieksekusi maka akan keluar error seperti itu,dan karena akses dari jenkins juga menggunakan tunnel yang mengarah ke ip 127 maka dari itu tls mengeluarkan issue karena 127 bukan scope
dari tls


### Solve

oke karena sudah ketemu akar dari masalahnya yaitu scope tls yang tidak mencakup ip dari 127 langsung saja masuk ke bagian eskekusinya untuk solusi sementara gunakan command

```
sudo -u jenkins kubectl config set-cluster kubernetes --insecure-skip-tls-verify=true
```

tapi untuk solusi permanen tinggal tambahkan saja extra sans untuk menambahkan scope dari tls,tinggal tambahkan di baris init didalam playbook-join.yaml

```
- name: init kubernetes
  hosts: masters
  become: yes
  tasks:
    - name: pre-pull kubernetes images
      ansible.builtin.command: kubeadm config images pull
      register: pull_output
    - name: init
      ansible.builtin.command: kubeadm init --apiserver-advertise-address=192.168.2.35 --pod-network-cidr=10.244.0.0/16 --node-name=node1 --apiserver-cert-extra-sans=127.0.0.1,192.168.50.106 <----- ini
      register: kube_output
      async: 600
      poll: 30
      ignore_errors: yes

    - name: ambil join command
      ansible.builtin.shell: kubeadm token create --print-join-command
      register: join_cmd_output

    - name: set fact join command
      set_fact:
        join_command: "{{ join_cmd_output.stdout }}"

- name: join worker ke master
  hosts: workers
  become: yes
  tasks:
    - name: join cluster
      ansible.builtin.shell: "{{ hostvars[groups['masters'][0]]['join_command'] }}"
```
