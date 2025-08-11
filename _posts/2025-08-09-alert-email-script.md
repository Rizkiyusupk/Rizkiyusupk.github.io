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

## Setup

Setelah semua installasi selesai sekarang bagian setup dan configurasi,pertama tama buat app password dari akun gmail yang akan digunakan
untuk referensi nya ada di sini silahkan baca 

[klik disini untuk referensinya](https://itsupport.umd.edu/itsupport?id=kb_article_view&sysparm_article=KB0015112)

setelah set up untuk akun gmail yang akan digunakan,selanjutnya membuat file dengan nama sasl_passwd di **/etc/postfix/sasl_passwd**

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo vim /etc/postfix/sasl_passwd
```
lalu tulis di dalamnya dengan format seperti dibawah ini

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
[smtp.gmail.com]:587  youremail@gmail.com:app_password
```

setelah itu kamu simpan,lalu jalankan command

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo postmap /etc/postfix/sasl_passwd
```

setelah itu berikutnya kamu konfigurasi di file **/etc/postfix/main.cf**,lalu ubah bagian bawah

> 💡 **Tips:** Pastikan menjalankan perintah ini sebagai **root** atau gunakan `sudo`.

```
sudo vim /etc/postfix/main.cf
```
lalu  ganti di bagian bawah **TLS** dengan confif berikut ini

```
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_tls_security_level=may

smtp_tls_CApath=/etc/ssl/certs
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
myhostname = ypurhostname
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
mydestination = $myhostname, yourhostname, localhost.localdomain, , localhost
relayhost = [smtp.gmail.com]:587
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all
# TLS settings
smtp_use_tls = yes
smtp_tls_security_level = encrypt
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt

# SASL authentication
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
```

setelah itu save


