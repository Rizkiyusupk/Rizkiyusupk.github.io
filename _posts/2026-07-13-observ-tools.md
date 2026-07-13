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

