---
title: "Pull Based Infrastructure Hub & Spoke "
date: 2026-10-22
infra_used: "Pull Based Argocd"
---

![asdjadvpisfb](/assets/images/hub_and_spoke/hb.png)

### Overview

Topik berikutnya benar-benar menguras tenaga dan pikiran saya,sebelumnya saya sudah pernah mengatakan jika saya ingin sekali menggunakan terraform namun saya tidak punya uang untuk sewa cloud ataupun menggunakan proxmox karena laptop 2 saya itu tidak mumpuni secara spesifikasi,makanya saya dengan tekad dan semangat saya yang membara mencara cara agar saya bisa belajar terraform, akhirnya setelah beberapa hari kebingungan harus bagaimana saya menemunkan video lama dari adinusa di youtube membahas tentang Terraform dan QEMU/KVM,saya tonton videonya dan saya menemukan resource akun github dan docs di terraform blogspot,topik kali ini menggabungkan beberapa tools seperti kubernetes,ansible,terraform,jenkins,QEMU/KVM,libvirt sebagai api,dll. Tidak usah berlama-lama lagi langsung gaskannnn

| Node        | CPU     | RAM  | Storage | Network                             |
|-------------|---------|------|---------|------------------------------------ |
| **Master**  | 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 1**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 2**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Jenkins** | 7 cores | 7GB  | 240GB   |                Wlan                 |
