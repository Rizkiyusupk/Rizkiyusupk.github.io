---
title: "Building a Lightweight Kubernetes Home Server with k3s (CLI-Only Setup) Using Bare Metal"
date: 2025-12-27
tags: [kubernetes, k3s, home-server, linux, cli, devops, containerd, nginx,home-server]
header:
  teaser: /assets/images/home-server/18555159.png
categories: [DevOps, Kubernetes,linux,Linux,Server]
---

![anjay](/assets/images/home-server/18555159.png)

## Overview

Pada dokumentasi ini, saya menjelaskan proses mengubah sebuah laptop lama yang sudah jarang digunakan menjadi single-node Kubernetes home server menggunakan k3s, tanpa bergantung pada virtual machine maupun antarmuka grafis (GUI).

Alih-alih menjalankan Kubernetes di dalam beberapa VM—yang menambah beban resource pada perangkat,dengan spesifikasi terbatas setup ini dijalankan langsung di bare metal dengan lingkungan Ubuntu CLI-only. Pendekatan ini bertujuan untuk mendapatkan sistem yang ringan, stabil, dan tetap mendekati kondisi production untuk skala rumahan.

## Software Version
- **OS: Ubuntu 25.04**
- **Kubectl: v1.35.0**
- **Docker: 28.2.2**


## Setup
Dikarenakan ini menggunakan bare metal dan bukan vm disini tidak ada setup vm seperti biasanya,jadi saya menggunakan laptop lama saya yang 
sudah jarang di gunakan sebagai single node dan kebetulan os dari laptop lama saya itu ubuntu jadinya saya bisa memanfaatkan laptop usang saya 
sebagai bahan belajar,dan kenapa saya memilih untuk k3s bukan k8s 
