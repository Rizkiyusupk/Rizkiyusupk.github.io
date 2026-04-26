---
title: "Debugging Error IaC: Building a Multi-Node Kubernetes Cluster from Scratch using Ansible"
date: 2026-04-26
tags: [kubernetes, linux, devops, crio,cri-o,system,infrastructure,Redhat,rhel]
header:
  teaser: /assets/images/iac/Untitled design (5).png
categories: [DevOps, Kubernetes,linux,Linux,Server,iac,infrastructure]
---

![asjdhb](/assets/images/iac/Untitled design (5).png)


### Overview 
Sampai sudah di postingan error seperti biasanya kali ini saya akan membuat postingan error khusus untuk membahas error apa saja yang saya temui ketika membangun projek iac
yang pertama, jika tidak tahu yang mana klik saja [disini](https://rizkiyusupk.github.io/devops/kubernetes/linux/server/iac/infrastructure/iac/) error yang akan dibahas 
error-error yang tidak general seperti "port already in use" ,jika sudah tidak usah berlama-lama mari langsung saja masuk ke pembahasannya 


### Error Backslash 

Pada saat running playbook untuk generate token dari kubernetes ada bagian pada playbook tersebut yang dimana di peruntukan untuk mengambil output dari kubeadm init supaya
bisa di simpan dan di gunakan agar node lain bisa join ke cluster yang sudah di buat, nah pada kasus ini output dari si kubeadm init itu memiliki backslash sebagai penyambung
baris contohnya seperti ini

```
kubeadm join 192.168.1.126:6443 --token abc.123 \
    --discovery-token-ca-cert-hash sha256:xyz...

```

bisa di liat sebagai contoh diatas ada backslash '\' di bagian baris pertama itu adalah output dari kubeadm init, python di ansbile kebingungan dengan backslash tersebut 
dan menangapnya sebagai special character, jadinya akan menimbulkan error message seperti itu


### Solve

Jika membaca dan mengikuti sesuai dengan tulisan yang ada di blog projek sebelumnya, bisa di pastikan bahwa error ini bisa di hindari, tapi jika kurang teliti 
playbook untuk generate dan join node itu harus mirip dengan ini

```
- name: init kubernetes
  hosts: master
  become: yes

  tasks:
    - name: init
      ansible.builtin.command: kubeadm init  --apiserver-advertise-address=192.168.1.xxx  --pod-network-cidr=10.244.0.0/16 --node-name=node1
      register: kube_output
      ignore_errors: yes
    - name: store the output
      ansible.builtin.copy:
        content: "{{ kube_output.stdout }}"
        dest: "/home/iki/token.txt"
    - name: Ambil join command secara bersih
      shell: echo "{{ kube_output.stdout }}" | grep -A 2 "kubeadm join" | tr -d '\\\n'
      register: clean_join
    - name: join command
      set_fact:
        join_command: "{{ clean_join.stdout }}"
- name: join
  hosts: slave1,slave2,
  become: yes

  tasks:
    - name: join command
      ansible.builtin.shell: "{{ hostvars['node1'].join_command }}"

```
 
nah cukup salin saja playbook ini, namu jika butuh penjelasan lebih jelas bisa langsung saja membaca blognya dengan seksama dan teliti jangan ada yang di skip ya!!


### Error No package kubelet available

heheh sedikit malu rasanya jika memasukan ini ke dalam error post, karena error ini murni kesalahan saya 😆,jadi contohnya seperti ini dibagian playbook untuk installasi
crio sebagai container runtime interface, pada task add repo kedalam node ada sedikit typo tapi sangat fatal jadi typonya seperti ini

```
  tasks:
    - name: add repo
      ansible.builtin.blockinfile:
        path: /etc/yums.repos.d/cri-o.repo
        block: |
          [cri-o]
          name=CRI-O
          baseurl=https://pkgs.k8s.io/addons:/cri-o:/stable:/{{ crio_version }}/rpm/
          enabled=1
          gpgcheck=1
          gpgkey=https://pkgs.k8s.io/addons:/cri-o:/stable:/{{ crio_version }}/rpm/repodata/repomd.xml.key
        create: yes

```

coba perhatikan baik-baik contoh playbook diatas adalah playbook yang salah , jika di perhatikan secara teliti seharusnya tidak ada huruf 's' di dalam kata 'yum'
karena ini menggunakan rhel dan dalam default penulisan di rhel itu

```
yum
```

bukannya

```
yums
```

jadinya tambahan satu huruf saja bisa berpengaruh besar terhadap keseluruhan playbook, jadinya jika di run maka akan muncul error seperti ini

```
fatal: [192.168.100.78]: FAILED! => {"changed": false, "failures": ["No package kubelet available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}

fatal: [192.168.100.77]: FAILED! => {"changed": false, "failures": ["No package kubelet available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}

fatal: [192.168.100.76]: FAILED! => {"changed": false, "failures": ["No package kubelet available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}


```

### Solve

Karena error nya hanya berupa typo tapi sangat fatal :v jadinya tinggal lebih berhati-hati saja dan langsung saja perbaiki playbook diatas hehe

### Missing Collection

Karena di beberapa playbook menggunakan  modul yang tidak ada secara default dan perlu installasi tambahan contohnya seperti posix maka dari itu error ini pastinya 
muncul,wajib hukumnya install terlebih dahulu collection posix karena jika tidak ada colletion yang di perlukan untuk suatu modul tertentu dan 
spesifik seperti posix contohnya akan ada error muncul seperti ini

```
The error appears to be in '/home/iki/playbook-xxx.yaml': line xx, column x, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

    - name: set SELINUX
      ansible.posix.selinux:
      ^ here

This error can be caused by:
1. The collection 'ansible.posix' is not installed.
2. The collection name is misspelled.
3. The collection is installed but not in the search path.
```

atau bisa juga dalam bentuk yang lebih simpel

```
couldn't resolve module/action 'ansible.posix.selinux'. This often indicates a misspelling, missing collection, or incorrect module path.
```

nah error diatas itu bisa terjadi karena colletion yang belum di install terlebih dahulu

### Solve

Karena akar masalahnya sudah di temukan maka tinggal perbaiki saja, langsung saja install collection yang dibutuhkan gunakan command

```
ansible-galaxy collection install ansible.posix
```

tunggu hingga proses installasi selesai lalu jika sudah tinggal run lagi playbooknya
