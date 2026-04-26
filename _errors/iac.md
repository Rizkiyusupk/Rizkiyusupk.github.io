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

### Error Missing Collection

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

### Error ansible builtin command

Untuk error ini itu cukup menarik menurut saya karena hanya berbeda sedikit dengan versi yang tidak errornya , di bagian join ada baris seperti ini 

```
    ansible.builtin.shell: "{{ hostvars['node1'].join_command }}"
```

dan jika baris itu di ganti dengan 

```
    ansible.builtin.command: "{{ hostvars['node1'].join_command }}"
```

maka akan ada error message seperti ini

```
fatal: [node2]: FAILED! => {"changed": false, "cmd": "'[init]' Using Kubernetes version: v1.30.14 '[preflight]' Running pre-flight checks '[preflight]' Pulling images required for setting up a Kubernetes cluster '[preflight]' This might take a minute or two, depending on the speed of your internet connection '[preflight]' You can also perform this action in beforehand using 'kubeadm config images pull' '[certs]' Using certificateDir folder /etc/kubernetes/pki '[certs]' Generating ca certificate and key '[certs]' Generating apiserver certificate and key '[certs]' apiserver serving cert is signed for DNS names '[kubernetes' kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local 'node1]' and IPs '[10.96.0.1' '192.168.100.76]' '[certs]' Generating apiserver-kubelet-client certificate and key '[certs]' Generating front-proxy-ca certificate and key '[certs]' Generating front-proxy-client certificate and key '[certs]' Generating etcd/ca certificate and key '[certs]' Generating etcd/server certificate and key '[certs]' etcd/server serving cert is signed for DNS names '[localhost' 'node1]' and IPs '[192.168.100.76' 127.0.0.1 '::1]' '[certs]' Generating etcd/peer certificate and key '[certs]' etcd/peer serving cert is signed for DNS names '[localhost' 'node1]' and IPs '[192.168.100.76' 127.0.0.1 '::1]' '[certs]' Generating etcd/healthcheck-client certificate and key '[certs]' Generating apiserver-etcd-client certificate and key '[certs]' Generating sa key and public key '[kubeconfig]' Using kubeconfig folder /etc/kubernetes '[kubeconfig]' Writing admin.conf kubeconfig file '[kubeconfig]' Writing super-admin.conf kubeconfig file '[kubeconfig]' Writing kubelet.conf kubeconfig file '[kubeconfig]' Writing controller-manager.conf kubeconfig file '[kubeconfig]' Writing scheduler.conf kubeconfig file '[etcd]' Creating static Pod manifest for local etcd in /etc/kubernetes/manifests '[control-plane]' Using manifest folder /etc/kubernetes/manifests '[control-plane]' Creating static Pod manifest for kube-apiserver '[control-plane]' Creating static Pod manifest for kube-controller-manager '[control-plane]' Creating static Pod manifest for kube-scheduler '[kubelet-start]' Writing kubelet environment file with flags to file /var/lib/kubelet/kubeadm-flags.env '[kubelet-start]' Writing kubelet configuration to file /var/lib/kubelet/config.yaml '[kubelet-start]' Starting the kubelet '[wait-control-plane]' Waiting for the kubelet to boot up the control plane as static Pods from directory /etc/kubernetes/manifests '[kubelet-check]' Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s '[kubelet-check]' The kubelet is healthy after 502.372625ms '[api-check]' Waiting for a healthy API server. This can take up to 4m0s '[api-check]' The API server is healthy after 4.505949826s '[upload-config]' Storing the configuration used in ConfigMap kubeadm-config in the kube-system Namespace '[kubelet]' Creating a ConfigMap kubelet-config in namespace kube-system with the configuration for the kubelets in the cluster '[upload-certs]' Skipping phase. Please see --upload-certs '[mark-control-plane]' Marking the node node1 as control-plane by adding the labels: '[node-role.kubernetes.io/control-plane' 'node.kubernetes.io/exclude-from-external-load-balancers]' '[mark-control-plane]' Marking the node node1 as control-plane by adding the taints '[node-role.kubernetes.io/control-plane:NoSchedule]' '[bootstrap-token]' Using token: wkf9l0.yg4qrbfd0hf1jsdh '[bootstrap-token]' Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles '[bootstrap-token]' Configured RBAC rules to allow Node Bootstrap tokens to get nodes '[bootstrap-token]' Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials '[bootstrap-token]' Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token '[bootstrap-token]' Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster '[bootstrap-token]' Creating the cluster-info ConfigMap in the kube-public namespace '[kubelet-finalize]' Updating /etc/kubernetes/kubelet.conf to point to a rotatable kubelet client certificate and key '[addons]' Applied essential addon: CoreDNS '[addons]' Applied essential addon: kube-proxy Your Kubernetes control-plane has initialized 'successfully!' To start using your cluster, you need to run the following as a regular user: mkdir -p /root/.kube sudo cp -i /etc/kubernetes/admin.conf /root/.kube/config sudo chown '$(id' '-u):$(id' '-g)' /root/.kube/config Alternatively, if you are the root user, you can run: export KUBECONFIG=/etc/kubernetes/admin.conf You should now deploy a pod network to the cluster. Run kubectl apply -f '[podnetwork].yaml' with one of the options listed at: https://kubernetes.io/docs/concepts/cluster-administration/addons/ Then you can join any number of worker nodes by running the following on each as root: kubeadm join 192.168.100.76:6443 --token wkf9l0.yg4qrbfd0hf1jsdh '\n' --discovery-token-ca-cert-hash sha256:0d15d3f5e971b868b825a53bdfd8c21b816ae32c5a993731d21138f06d9ee24a '|' grep -A 2 kubeadm join '|' tr -d '\\n'", "msg": "[Errno 2] No such file or directory: b'[init]'", "rc": 2, "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
fatal: [node3]: FAILED! => {"changed": false, "cmd": "'[init]' Using Kubernetes version: v1.30.14 '[preflight]' Running pre-flight checks '[preflight]' Pulling images required for setting up a Kubernetes cluster '[preflight]' This might take a minute or two, depending on the speed of your internet connection '[preflight]' You can also perform this action in beforehand using 'kubeadm config images pull' '[certs]' Using certificateDir folder /etc/kubernetes/pki '[certs]' Generating ca certificate and key '[certs]' Generating apiserver certificate and key '[certs]' apiserver serving cert is signed for DNS names '[kubernetes' kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local 'node1]' and IPs '[10.96.0.1' '192.168.100.76]' '[certs]' Generating apiserver-kubelet-client certificate and key '[certs]' Generating front-proxy-ca certificate and key '[certs]' Generating front-proxy-client certificate and key '[certs]' Generating etcd/ca certificate and key '[certs]' Generating etcd/server certificate and key '[certs]' etcd/server serving cert is signed for DNS names '[localhost' 'node1]' and IPs '[192.168.100.76' 127.0.0.1 '::1]' '[certs]' Generating etcd/peer certificate and key '[certs]' etcd/peer serving cert is signed for DNS names '[localhost' 'node1]' and IPs '[192.168.100.76' 127.0.0.1 '::1]' '[certs]' Generating etcd/healthcheck-client certificate and key '[certs]' Generating apiserver-etcd-client certificate and key '[certs]' Generating sa key and public key '[kubeconfig]' Using kubeconfig folder /etc/kubernetes '[kubeconfig]' Writing admin.conf kubeconfig file '[kubeconfig]' Writing super-admin.conf kubeconfig file '[kubeconfig]' Writing kubelet.conf kubeconfig file '[kubeconfig]' Writing controller-manager.conf kubeconfig file '[kubeconfig]' Writing scheduler.conf kubeconfig file '[etcd]' Creating static Pod manifest for local etcd in /etc/kubernetes/manifests '[control-plane]' Using manifest folder /etc/kubernetes/manifests '[control-plane]' Creating static Pod manifest for kube-apiserver '[control-plane]' Creating static Pod manifest for kube-controller-manager '[control-plane]' Creating static Pod manifest for kube-scheduler '[kubelet-start]' Writing kubelet environment file with flags to file /var/lib/kubelet/kubeadm-flags.env '[kubelet-start]' Writing kubelet configuration to file /var/lib/kubelet/config.yaml '[kubelet-start]' Starting the kubelet '[wait-control-plane]' Waiting for the kubelet to boot up the control plane as static Pods from directory /etc/kubernetes/manifests '[kubelet-check]' Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s '[kubelet-check]' The kubelet is healthy after 502.372625ms '[api-check]' Waiting for a healthy API server. This can take up to 4m0s '[api-check]' The API server is healthy after 4.505949826s '[upload-config]' Storing the configuration used in ConfigMap kubeadm-config in the kube-system Namespace '[kubelet]' Creating a ConfigMap kubelet-config in namespace kube-system with the configuration for the kubelets in the cluster '[upload-certs]' Skipping phase. Please see --upload-certs '[mark-control-plane]' Marking the node node1 as control-plane by adding the labels: '[node-role.kubernetes.io/control-plane' 'node.kubernetes.io/exclude-from-external-load-balancers]' '[mark-control-plane]' Marking the node node1 as control-plane by adding the taints '[node-role.kubernetes.io/control-plane:NoSchedule]' '[bootstrap-token]' Using token: wkf9l0.yg4qrbfd0hf1jsdh '[bootstrap-token]' Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles '[bootstrap-token]' Configured RBAC rules to allow Node Bootstrap tokens to get nodes '[bootstrap-token]' Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials '[bootstrap-token]' Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token '[bootstrap-token]' Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster '[bootstrap-token]' Creating the cluster-info ConfigMap in the kube-public namespace '[kubelet-finalize]' Updating /etc/kubernetes/kubelet.conf to point to a rotatable kubelet client certificate and key '[addons]' Applied essential addon: CoreDNS '[addons]' Applied essential addon: kube-proxy Your Kubernetes control-plane has initialized 'successfully!' To start using your cluster, you need to run the following as a regular user: mkdir -p /root/.kube sudo cp -i /etc/kubernetes/admin.conf /root/.kube/config sudo chown '$(id' '-u):$(id' '-g)' /root/.kube/config Alternatively, if you are the root user, you can run: export KUBECONFIG=/etc/kubernetes/admin.conf You should now deploy a pod network to the cluster. Run kubectl apply -f '[podnetwork].yaml' with one of the options listed at: https://kubernetes.io/docs/concepts/cluster-administration/addons/ Then you can join any number of worker nodes by running the following on each as root: kubeadm join 192.168.100.76:6443 --token wkf9l0.yg4qrbfd0hf1jsdh '\n' --discovery-token-ca-cert-hash sha256:0d15d3f5e971b868b825a53bdfd8c21b816ae32c5a993731d21138f06d9ee24a '|' grep -A 2 kubeadm join '|' tr -d '\\n'", "msg": "[Errno 2] No such file or directory: b'[init]'", "rc": 2, "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}

```

jika mengikuti playbook yang ada di blog projek, di baris itu seharusnya memang shell, terus kenapa bisa shell sedangkan baris-baris sebelumnya itu 
command bukan shell? karena command tidak bisa membedakan mana teks biasa dengan spesial character ataupun wildcard, jika saja di baris itu memang shell maka node
akan membuka shell benar benar sebuah bash shell, jika saja itu command maka node akan menanggap itu sebagai sebuah aplikasi dan malah ingin melakukan [init] dari 
argumen yang sudah di simpan di variable, jadinya ansible kebingungan karena command tidak bisa membedakan mana spesial character ataupun wildcard

### Solve

Jika benar benar mengikuti sesuai dengan instruksi yang ada di blog,maka seharusnya error seperti ini bisa di hindari, gunakan shell seperti yang di contohkan di blog

