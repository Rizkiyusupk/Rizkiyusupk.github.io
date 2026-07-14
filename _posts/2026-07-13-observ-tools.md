---
title: "Designing a Cross-Platform DevOps Lab: From Provisioning to Production-Grade Observability (K8s, Terraform, Ansible, Prometheus, Loki, Alertmanager)"
date: 2026-07-13
tags: [kubernetes, linux, devops,containerd,system,infrastructure,Ci/Cd,KVM,QEMU,Terraform,terraform,Ansible,Loki.Prometheus,]
header:
  teaser: /assets/images/observ/INFRASTUCTURE (2).png
categories: [DevOps, Kubernetes,linux,Linux,Server,iac,infrastructure]
---
![adifjgby](/assets/images/observ/INFRASTUCTURE (2).png)

### Overview

Masuk ke tahap lanjutan dari projek sebelumnya jika di projek sebelumnya itu belum menambahkan observ tools untuk monitoring dari infrastructure yang sudah di bangun sekarang saya akan membahas
cara untuk menambahkan stack observ tools seperti prometheus,loki,grafana,dan alertmanager.Karena kali ini hanya menambahkan dan mengintegrasikan observ tools jadinya untuk infrastructure 
itu masih sama saja dengan projek sebelumnya,oke langsung saja masuk ke pembahasaanya 


| Node        | CPU     | RAM  | Storage | Network                             |
|-------------|---------|------|---------|------------------------------------ |
| **Master**  | 1 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 1**| 1 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 2**| 1 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Jenkins** | 7 cores | 7GB  | 240GB   |                Wlan                 |

![scdbijcf](/assets/images/observ/ChatGPT Image Jul 13, 2026, 06_53_45 PM.png)


### Structure Folder 

Untuk structure folder yang digunakan dalam projek ini ada dua yang pertama itu untuk terraform dan yang kedua itu ansible

```
terraform-setup/
├── .terraform/
├── compute.tf
├── main.tf
├── prep-vm.tf
├── terraform.tfstate
└── terraform.tfstate.backup
```

dan yang kedua yaitu ansible

```
k8s/
├── ansible.cfg
├── inventory
├── playbook-allow-port.yaml
├── playbook-enable-service-baremetal.yaml
├── playbook-ip.yaml
├── playbook-install-java-baremetal.yaml
├── playbook-install-jenkins-baremetal.yaml
├── playbook-install-kubectl-baremetal.yaml
├── playbook-join.yaml
├── playbook-kubernetes.yaml
├── playbook-pkg.yaml
├── playbook-swap.yaml
├── observ.yaml
├── observ_2.yaml
```

### Tools

- **OS Laptop 1** : Windows 11 Pro
- **OS WSL2 Subsystem** : Ubuntu 24.04 LTS
- **OS Ubuntu VM (K8s Nodes)** : 24.04 LTS
- **OS Ubuntu Jenkins Node (Bare Metal)** : 25.04
- **Kubernetes** : 1.28
- **Containerd** : 2.2.4
- **Java** : 21
- **Jenkins** : 2.56
- **Network** : Flannel
- **Git Bash** : 2.51.1
- **Terraform** :  v1.15.6
- **Ansible** : v2.16+
- **Helm** : v3.x
- **Telegram Bot** :
- **KVM** : 8.2.2
- **QEMU** : 8.2.2
- **Libvirt** : 10.0.0
- **Alertmanager** : v0.33.1
- **Loki** : 3.6.7
- **Loki Canary** : 3.6.7
- **Loki Gateway (nginx)** : 1.29-alpine
- **Grafana** : 13.1.0
- **Prometheus Operator** : v0.92.1
- **Prometheus Config Reloader** : v0.92.1
- **Kube State Metrics** : v2.19.1
- **Prometheus** : v3.13.1
- **Node Exporter** : v1.12.0
- **Promtail** : 3.5.1

  ### Setup

  Oke masuk ke tahap pertama dari proses ini yaitu setup helm untuk melakukan installasi plg stack pertama-tama buat sebuah file playbook

  ```
  vim observ.yaml
  |
  - name: install observ tech stack
  hosts: masters
  become: true

  tasks:
    - name: install helm
      ansible.builtin.shell:    curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
    - name: add repo for node-exporter
      ansible.builtin.shell: helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    - name: update package
      ansible.builtin.shell: helm repo update
    - name: install package
      ansible.builtin.shell: helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring  --kubeconfig /home/ubuntu/.kube/config --create-namespace
  ```

gunakan command

```
ansible-playbook playbook.yaml
|
ansible-playbook observ.yaml
```

playbook diatas melakukan installasi untuk helm dan untuk melakukan semua installasi dari plg stack yang akan kita gunakan nantinya,setelah melakukan installasi dari helm task selanjutnya 
yaitu melakukan installasi dari node exporter untuk mengambil semua metrics dan data dari setiap node dan node exporter berlaku sebagai daemon set di semua node yang nantinya di 
distribusikan oleh master node sekalian untuk installsi dari promeheues dan untuk task terakhir yaitu untuk helm installasi dan memnbuat sebuah namespace dengan nama 
monitoring,selanjutnya yaitu instlaasi dari loki,grafana

```
vim observ_2.yaml
|
- name: install loki stack
  hosts: masters
  become: true
  tasks:
    - name: add repo grafana
      ansible.builtin.shell: helm repo add grafana https://grafana.github.io/helm-charts
    - name: update package
      ansible.builtin.shell: helm repo update
    - name: install loki monolithic
      ansible.builtin.shell: >
        helm install loki grafana/loki
        --namespace monitoring
        --kubeconfig /home/ubuntu/.kube/config
        --set deploymentMode=SingleBinary
        --set loki.commonConfig.replication_factor=1
        --set loki.storage.type=filesystem
        --set loki.schemaConfig.configs[0].from=2024-04-01
        --set loki.schemaConfig.configs[0].store=tsdb
        --set loki.schemaConfig.configs[0].object_store=filesystem
        --set loki.schemaConfig.configs[0].schema=v13
        --set loki.schemaConfig.configs[0].index.prefix=loki_index_
        --set loki.schemaConfig.configs[0].index.period=24h
        --set singleBinary.replicas=1
        --set singleBinary.persistence.enabled=true
        --set singleBinary.persistence.size=5Gi
        --set read.replicas=0
        --set write.replicas=0
        --set backend.replicas=0

    - name: install promtail
      ansible.builtin.shell: >
        helm install promtail grafana/promtail
        --namespace monitoring
        --kubeconfig /home/ubuntu/.kube/config
        --set config.clients[0].url=http://loki:3100/loki/api/v1/push
```

gunakan command 

```
ansible-playbook
|
playbook observ_2.yaml
```

untuk task pertama yaitu menambahkan repo resmi dari grafana lalu melakukan update repo menggunakan helm lalu,di task selanjutnya yaitu melakukan installasi loki menggunakan helm disini 
saya menggunakan loki monolithic karena saya membuat infrastructure skala homelab dengan resource terbatas jadinya saya tidak ingin membuat beban terhadap laptop saya dengan menggunakan
loki tipe lain jika saja saya meggunakan loki tipe scalable atau microservices saya akan membuat beban yang cukup besar dengan resource yang saya punya sekarang,tapi meskipun resource 
saya terbatas saya tetap membangun sebuah infrastructure yang production grade, dan yang terakhir di task terakhir itu melakukan installasi dari promtail,agar nantinya bisa collect log,
tunggu hingga proses installasi selesai jika sudah jangan langsung masuk ke browser untuk mengakses si dashboard karena tidak bisa langsung mengakses dashboard dibutuhkan tunneling dan
port fordward terlebih dahulu tapi sebelum itu buka wsl dan install socat terlebih dahulu gunakan command

```
apt-get install -y socat
```

oke karena socat sudah terinstall lalu selanjutnya gunakan command ini

```
di node-master
|
kubectl port-forward svc/prometheus-grafana 3001:80 -n monitoring --address=0.0.0.0 &
```

dan

```
di wsl
|
socat TCP-LISTEN:8884,bind=0.0.0.0,fork TCP:10.10.10.55:3001 &
```

nah kenapa harus sekali melakukan tunneling dan port forward seperti ini karena layer dari infrastructure kali ini itu benar benar cukup rumit untuk user agar langsung bisa mengakses 
semua dashboard,jadinya disini perlu membypass semua network layer yang ada dan nantinya untuk memeriksa alertmanager juga perlu bypass sekali lagi oke jika sudah bisa langsung di cek 

```
http://localhost:8884
```

jika sudah seharusnya akan muncul untuk opsi login

![asdosvfuh](/assets/images/observ/memek.png)

oke jika sudah ada tampilan login seperti ini masukan 

```
username = admin
```

dan untuk passowrd nya gunakan output dari command berikut

```
kubectl get secret prometheus-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 -d
echo
```

jika sudah akan di bawa ke dashboard dari si prometheus seperti ini

![sadsifjuh](/assets/images/observ/Screenshot 2026-07-14 193250.png)

oke jika sudah berjalan dengan sempurna

