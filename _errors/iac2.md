---
title: "Debugging IaC: Vagrant as Provisioning For Building a Kubernetes Cluster"
date: 2026-05-17
tags: [kubernetes, linux, devops, crio,cri-o,system,infrastructure,]
header:
  teaser: /assets/images/iac2/Untitled design (6).jpg
categories: [DevOps, Kubernetes,linux,Linux,Server,iac,infrastructure]
---

![aidasd](/assets/images/iac2/Untitled design (6).jpg)

### Overview 
Pada postingan error kali ini saya akan membahas error-error apa saja yang saya temui pada saat membuat projek IaC: Vagrant as Provisioning For Building a Kubernetes Cluster,langsung saja 
masuk ke pembahasanya

### Shared Folder Tidak terbaca

Ketika melakukan proses pembuatan golden image menggunakan virtualbox,dalam proses setup vm dibutuhkan ssh key dari vagrant karena mekanisme dari vagrant itu sendiri mirip seperti ansible
vagrant bisa berjalan sebagai provisioning menggunakan ssh dan bisa mengakses semua node ataupun vm yang sudah di buat menggunakan ssh,maka dari itu sebelum masuk ke cara kerja dari vagrant
ssh key dari vagrant harus di install terlebih dahulu dan karena ssh key itu diambil dari repo github dan memiliki  url yang panjang dan rumit maka salah satu solusi dengan menggunakan 
salah satu fitur yang ada di virtualbox yaitu shared folder nah shared folder nantinya berguna ketika membuat file script yang dimana salah satu baris di script itu berisi install ssh key
vagrant dari repo resminya jadinya semua akan aman tapi ada masalah dimana shared folder tidak terbaca oleh vm ketika di cek di 

```
/media/
```

biasanya ubuntu akan menaruh nya disana namun jika ada masalah shared folder tidak akan terbaca

### Solve

Untuk Solusi dari itu maka perlu dilakukan installasi virtualbox guest addtions karena jika ingin menggunaka fitur shared folder yang ada di virtual box perlu guest addtional terinstall dan
berjalan jika ingin fitur tersebut bisa berfungsi,maka dari itu perlu di lakukan installasi agar error ini bisa hilang gunakan command

```
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
```

### File Format Windows

Dalam Proses setup ada tahapan setup golden image ada tahapan dimana proses setup melibatkan shared folder yang tadi sudah di bahas sebelumnya jika shared folder sudah terbaca dan terdapat
file script sh yang sudah di buat di windows untuk setup golden imagenya file tersebut akan memiliki format aneh yang tidak bisa di eksekusi oleh linux karena pembuatan sebelumnya itu 
di buat di windows dan bukannya di linux,akan ada format dengan huruf M besar di setiap akhir baris script kurang lebih seperti ini

![aoshda](/assets/images/iac2/Screenshot 2026-05-11 125655.png)

jika ada output seperti itu maka bisa di pastikan penyebabnya ialah format windows 

### Solve

Jika sudah ditemukan biang keroknya maka tinggal eksekusi saja gunakan command

```
sed -i 's/\r$//' namafile.sh
```

untuk merubah format file 

### Terminal Stuck 

Masuk ke tahap setup kubernetes ada playbook yang berfungsi untuk melakukan installasi gpg key dari kubernetes yang nantinya baris itu akan mengambil key lalu menambahkan key tersebut ke 
direktori node kurang lebih seperti ini detail barisnya

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

nah untuk proses installasi kubernetes biasa memang begini tapi karena dalam pembahasan menggunakan ansible sebagai iac jadinya akan ada sedikit perbedaan,jika tetap menggunakan baris
seperti installasi normal maka hanya akan ada error yang dimana terminal dari vscode akan stuck dan tidak ada proses lainnya 

### Solve

untuk solve error ini kamu hanya perlu menambahkan baris berikut

```
--batch --yes
```

baris itu jadinya akan menghilangkan interactive terminal yang akan membuatnya otomatis menjawab yes,jadinya full baris itu akan seperti

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor --batch --yes -o /usr/share/keyrings/docker-archive-keyring.gpg
```
