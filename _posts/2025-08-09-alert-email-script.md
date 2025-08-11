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
- Smtp untuk mengirim alert ke email di sini kita bakalan pakai postfix dan Mutt buat attach file output dari script 
- Akun Gmail untuk SMTP (atau sesuaikan dengan SMTP lain)

## Instalasi dan Setup
Langkah pertama dalam pembuatan script ini ialah installasi smtp dan mutt,untuk itu lebih baiknya update package terlebih dahulu

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo apt-get update
```

setelah kamu menupdate package sistem mu langsung saja install postfixnya

```
sudo apt install postfix -y
```

kenapa pakai **-y** karena agar bisa langsung menjawab pertanyaan **y/n?** saat penginstalan berlangsung

![Logo](/assets/images/automation_alert_email/Screenshot 2025-08-11 134044.png)

jika ada tampilan seperti ini pilih site internet,selanjutnya jika ada tampilan seperti ini

![logo1](/assets/images/automation_alert_email/Screenshot 2025-08-11 134105.png)

isi dengan nama hostname mu,lalu setelah itu akan ada tampilan untuk reboot seperti ini
tekan **enter** untuk merboot tinggalkan semua dalam kondisi default

![Logo2](/assets/images/automation_alert_email/Screenshot 2025-08-11 134818.png)

setelah itu sistem akan mengalami reboot tunggu saja beberapa saat.

lalu setelah sistem reboot langsung install mailutilsnya

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo apt-get mailutils -y
```

seperti biasa pakai flag **-y** di saat penginstallan akan ada opsi untuk reboot lagi seperti di postfix tadi

![Logo2](/assets/images/automation_alert_email/Screenshot 2025-08-11 134818.png)

tekan **enter** lagi untuk mereboot

##Setup
Setelah semua installasi selesai sekarang bagian setup dan configurasi,pertama tama buat app password dari akun gmail yang akan digunakan
untuk referensi nya ada di sini silahkan baca https://support.google.com/mail/answer/185833?hl=en

