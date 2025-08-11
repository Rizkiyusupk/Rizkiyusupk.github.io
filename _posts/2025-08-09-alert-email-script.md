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
lalu  ganti di bagian bawah **TLS** dengan config berikut ini

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

setelah itu save,setelah itu coba untuk test mail terlebih dahulu

```
echo "test" | mail -s "test" youremail@gmail.com
```
ini adalah contoh jika testnya berhasil 

![gambar](/assets/images/automation_alert_email/WhatsApp Image 2025-08-11 at 19.13.27.jpeg)

🥳🥳🥳🥳🥳🥳🥳🥳🥳🥳🥳🥳🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀🚀
## Scripting

nah sekarang sudah sampai dibagian dimana script akan dibuat,**spoiler!!** saya disini akan menggunakan
awk sebagai command yang akan mengambil nilai atau value dan memasukan nya ke dalam script

>💡 **Tips:** jalankan command dibawah ini agar script nantinya bisa jalan


```
docker run --rm -it polinux/stress stress --cpu 4
dd if=/dev/zero of=dummyfile bs=1M count=10240
```

command diatas berfungsi sebagai pemberat untuk sistem,karena ini adalah script yang mendeteksi adanya penggunaan resource maka dari itu butuh pemberat
🏋️🏋️🏋️🏋️🏋️🏋️🏋️
selanjutnya kamu tinggal membuat scriptnya langkah awal pembuatanya,buat file terlebih dahulu kamu bisa membuatnya dengan **touch** ataupun **echo** [referensi untuk command command linux](https://linuxcommand.org/lc3_man_page_index.php) bisa juga langsung dengan command text editor
seperti **vim** [refrensi untuk vim](https://www.vim.org/docs.php) ataupun **nano** [referensi untuk nano](https://docs.nano.org/)

jika dengan command akan seperti ini
> 💡 **Tips:** format dari script itu wajib **.sh**


```
touch namafile.sh
```

bukan hanya itu saja

```
echo "isi text" > namafile.sh
```

tapi sedikit berbeda dengan text editor karena jika membuat file menggunakan text editor maka kita akan langsung masuk ke menu text editor itu sendiri
contoh

```
vim namafile.sh
```

![logo3](/assets/images/automation_alert_email/Screenshot 2025-08-11 195056.png)

lalu jika kamu pakai nano 
```
nano namafile.sh
```

![logo4](/assets/images/automation_alert_email/Screenshot 2025-08-11 195108.png)

setelah membuat filenya bisa langsung masuk ke tahap pembuatan scriptnya,langkah awal dalam scripting ialah pembuatan shebang
bisa kamu lihat apa itu shebang di sini [referensi](https://en.wikipedia.org/wiki/Shebang_(Unix))

>💡 **Tips:** disini karena kita akan pakai bash jadi pakai /bin/bash

```
#!/bin/bash
```

setelah shebang sudah dibuat langsung masuk ke bagian utama scriptnya,sudah saya sebut sebelumnya script ini akan menggunakan awk
berikut [referensi awk](https://www.geeksforgeeks.org/linux-unix/awk-command-unixlinux-examples/)

>💡 **Tips:** disini saya juga memakai while loop agar script bisa jalan terus di latar belakang supaya nanti waktu script di jalankan bisa menggunakan cronjob agar bisa mengotomatisasi tugas,dan nantinya script akan mengirim alert terus menerus,tulis dibawah shebang tadi

```
while true
do
        system="$(top -b -n1 | awk '{if ($9 > 60 && $12 == "stress") { print "penggunaan cpu diatas batas wajar:", $9 }}' > output.txt)"
        system1="$(df -h | awk '{usage = substr ($5,1 ,length($5)-1; if (usage+0 > 10 ) print "pengunaan disk diatas batas wajar:"$5}' >> output.txt)"
        echo "Ini hasil monitoring CPU dan Disk." | mutt -s "Laporan Monitoring" -a output.txt -- example@gmail.com
done
```

Berikut penjelasan dari dua line diatas, kenapa saya memakai sebuah variable karena saya ingin tidak ada output yang memenuhi layar jika kamu coba manual top -b -n1 trus pakai awk tadi 
maka akan ada output yang cukup menganggu,dan di awk saya memakai dua variable yaitu **$9** & **$12**, variable **$9** itu sendiri merupakan kolom cpu dimana penggunaan cpu oleh program di tampilkan lalu variable **12** adalah nama program yang sudah kita set sebelumnya lalu ada redirect ke file bernama output.txt untuk menyimpan output dari scriptnya lalu dikirim ke email.Untuk line kedua kita bakalan menggunakan dummy file sebagai pemberat dengan size 10gb agar sistem mendeteksi penggunaan dari memory yang melebihi 10% ,sebenarnya script nya sudah bisa jalan namun saya ingin menambahkan beberapa sentuhan,


```
while true
do
        echo "pengecekan sistem..."
        sleep 5
        system="$(df -h | awk '{usage = substr ($5,1, length($5)-1); if (usage+0 > 10) print "pengunaan disk diatas batas wajar:"$5}' > output.txt )"
        echo "pengiriman laporan melalui email.."
        sleep 5
        system1="$(top -b -n1 | awk '{ if ($9 > 60 && $12 == "stress") { print "penggunaan cpu diatas batas wajar:", $9 }}' >> output.txt)"
        echo "Ini hasil monitoring CPU dan Disk." | mutt -s "Laporan Monitoring" -a output.txt -- example@gmail.com
        echo "done"
        break
done
```

nah sipp kurang lebih seperti ini guys, saat nya test

<video controls width="640" height="360">
  <source src="{{ 'Rizkiyusupk.github.io/assets/video/output.mp4' | relative_url }}" type="video/mp4">
  Browser kamu tidak mendukung video tag.
</video>

nahh kurang lebih seperti ini outputnya jika sudah ada alert di email

![logo7](/assets/images/automation_alert_email/video/jembut_ketilang.jpeg)

dan seperti ini output dari file nya


![logo5](/assets/images/automation_alert_email/video/WhatsApp Image 2025-08-11 at 23.07.44.jpeg)

