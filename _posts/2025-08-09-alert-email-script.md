---
title: "Automation Alert Email Script Documentation"
date: 2025-08-09
tags: [automation, email, alert, script]
header:
  teaser: /assets/images/png-clipart-bash-shell-command-line-interface-ls-shell-rectangle-logo-thumbnail-removebg-preview.png

---
![Shell Scripting Logo](/assets/images/png-clipart-bash-shell-command-line-interface-ls-shell-rectangle-logo-thumbnail-removebg-preview.png)
# Automation Alert Email Script

## Deskripsi
Apa Itu Shell Scripting?
Shell scripting adalah proses menulis sekumpulan perintah yang dijalankan oleh shell, yaitu program yang menyediakan antarmuka command-line untuk berinteraksi dengan sistem operasi, terutama di lingkungan Unix/Linux. Script ini biasanya berisi serangkaian perintah yang dieksekusi secara berurutan, mirip dengan program kecil yang otomatisasi tugas-tugas yang sering dilakukan.

Dengan shell scripting, kamu bisa mengotomatisasi berbagai pekerjaan seperti mengelola file, menjalankan program, memonitor sistem, hingga konfigurasi server. Shell scripting sangat berguna untuk mempercepat workflow dan mengurangi kesalahan yang terjadi jika perintah dijalankan manual satu per satu.

Contoh shell scripting yang populer adalah menggunakan Bash (Bourne Again SHell), yang merupakan shell standar di banyak distribusi Linux.

Script ini berfungsi mengirimkan email alert otomatis untuk memantau kondisi server seperti disk usage dan load CPU.

## Prasyarat

- Ubuntu os (bisa memakai Virtual Mesin)
- Smtp untuk mengirim alert ke email dan Mutt buat attach file output dari script 
- Akun Gmail untuk SMTP (atau sesuaikan dengan SMTP lain)

## Instalasi dan Setup
