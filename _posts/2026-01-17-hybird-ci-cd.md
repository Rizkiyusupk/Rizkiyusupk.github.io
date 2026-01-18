---
title: "Build Ci/Cd Hybrid System Full Infrastructure Setup(Vm + Bare Metal)"
date: 2026-01-17
tags: [kubernetes, linux, cli, devops, containerd,bare-metal,hybrid,ci,cd,ci/cd,system,infrastructure]
header:
  teaser: /assets/images/hybird-ci-cd/Untitled design (1).jpg
categories: [DevOps, Kubernetes,linux,Linux,Server]
---

![logo1](/assets/images/hybird-ci-cd/jembod.jpg)

## Overview 

Pada post kali ini saya akan mendemonstrasikan step by step setup infrastruktur dari sebuah sistem ci/cd,
sama seperti di post sebelumnya soal yang [ini](https://rizkiyusupk.github.io/devops/kubernetes/Build-Ci-Cd-System/)
di post ini perbedaanya hanyalah pada topologi,jika di post sebelumnya itu full vm dipost kali ini saya akan melakukannya 
dengan tambahan 1 bare metal yaitu sebuah laptop.Bare metal disini itu berfungsi sebagai tambahan pembalajaran dan 
menambah kerumitan dari topologi sebelumnya, yang akan di gantikan bare metal yaitu vm jenkins yang sebelumnya berupa sebuah
vm di gantikan menjadi bare metal.Pada pratiknya ini nantinya akan melibatkan banyak sekali tools-tools dan beberapa,untuk topologi masih 
sama dengan yang lama cuman ada beberapa koreksi mengenai topologi seperti penambahan bare metal jenkins.Sementara untuk workflow itu
masih sama dengan yang sebelumnya jadi,cara kerja atau workflownya sama dengan post sebelumnya.
Berikut workflow dari project ini

![logoo1](/assets/images/ci-cd/2.png)
