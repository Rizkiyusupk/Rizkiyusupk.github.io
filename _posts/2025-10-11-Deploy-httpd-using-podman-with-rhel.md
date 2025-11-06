---
title: "Deploy HTTPD Container using podman in RED HAT ENTERPRISE LINUX(OpenShift Training Edition)"
date: 2025-11-06
tags: [podman, rhel, httpd, container, openshift, devops]
header:
  teaser: /assets/images/redhat/Red_Hat-Logo.wine.png
---
![logo](/assets/images/redhat/Red_Hat-Logo.wine.png)

Dalam post ini, saya akan membahas bagaimana cara melakukan **deployment HTTPD (Apache Web Server)** menggunakan **Podman** di sistem operasi **Red Hat Enterprise Linux (RHEL)**.  
Topik ini juga merupakan bagian dari latihan yang saya jalani dalam **program pelatihan OpenShift di Red Hat**.

**Podman** adalah tool manajemen container yang serupa dengan Docker, namun tanpa daemon dan lebih aman karena tidak memerlukan root privilege.  
Podman digunakan secara luas di lingkungan Red Hat dan OpenShift untuk membangun serta menjalankan container secara lokal.

Dengan Podman, kita dapat:
- Menjalankan container sebagai bagian dari sistem micro services
- Mempraktekan bagaimana cara container bekerja 
- Menjalankan container tanpa root 
- Mengelola **pods**


sebelumnya ini adalah gambar dari arsitektur sebuah container

![logo1](/assets/images/redhat/docker-containerized-and-vm-transparent-bg.png)

Nah perbedaan dari kedua arsitektur tersebut yaitu container dan virtual machine terletak di mana berjalannya sistem container dan virtual machine itu,
jika di perhatikan dengan lebih seksama bisa di lihat jika container akan langsung berjalan di atas sebuah operating sistem,yaitu di host operating sistem, sedangkan untuk virtual machine sendiri harus menginstall guest os terlebih dahulu sebelum di antarkan ke production,dengan kata lain container bisa langsung berjalan di operating sistem server sedangkan untuk virtual machine harus menginstall operating system tambahan untuk bisa berjalan di sebuah server atau di producion,
untuk lebih jelasnya lagi ini contohnya untuk docker
![logo3](/assets/images/redhat/applsci-12-06737-g001.png)

tapi di sini saya akan menggunakan **podman** sama seperti docker podman juga merupakan tools untuk micro service dan containerization,tidak jauh berbeda dengan docker podman juga merupakan 
tool yang populer untuk micro services atau container,dan juga saya disini akan mendemokan deploymen httpd menggunakan podman di RHEL(Red Hat Entterprise Linux),dikarenakan saya masuk
ke program openshift di rhel tersebut,

![logo](/assets/images/redhat/podman-logo-full-vert.png)

nah apa perbedaan podman dan docker?? kurang lebih seperti ini

| 🔧 Aspek / Fitur | **Podman** | **Docker** |
|------------------|-------------|-------------|
| **Arsitektur** | Tidak memakai daemon (*daemonless*). Setiap perintah berjalan langsung di sistem. | Menggunakan *Docker Daemon* (`dockerd`) sebagai background service untuk mengelola container. |
| **Privilege (Hak Akses)** | Dapat dijalankan **tanpa root** (*rootless containers*). | Biasanya memerlukan hak akses **root** untuk menjalankan daemon. |
| **Keamanan** | Lebih aman karena mode rootless bawaan. Tidak ada proses daemon dengan hak istimewa. | Relatif kurang aman karena daemon berjalan dengan hak istimewa sistem. |
| **Kompatibilitas Perintah** | 100% kompatibel dengan perintah `docker` (misalnya `podman run`, `podman ps`). | Native command (`docker run`, `docker ps`). |
| **Pods Support** | Mendukung konsep **Pod** seperti di Kubernetes (beberapa container dalam satu pod). | Tidak mendukung konsep “pod” secara langsung. |
| **Layanan Background (Daemon)** | Tidak punya daemon yang selalu berjalan di background. | Semua container dikontrol lewat satu daemon (`dockerd`). |
| **Integrasi Systemd** | Terintegrasi langsung dengan `systemd`, dapat membuat unit service container. | Butuh konfigurasi manual untuk integrasi `systemd`. |
| **Konfigurasi Registry** | Menggunakan `/etc/containers/registries.conf` untuk registry container. | Menggunakan `/etc/docker/daemon.json` untuk konfigurasi registry. |
| **Build Engine** | Menggunakan **Buildah** di belakang layar untuk membangun image. | Menggunakan build engine internal Docker. |
| **Ekosistem / Vendor** | Bagian dari ekosistem **Red Hat** (Podman, Buildah, Skopeo). | Dikembangkan oleh **Docker Inc.** dan didukung luas oleh komunitas open source. |
| **Kompatibilitas Kubernetes** | Bisa ekspor container jadi YAML Kubernetes dengan `podman generate kube`. | Perlu konfigurasi tambahan (tidak otomatis). |
| **Dukungan OpenShift / RHEL** | Didukung penuh oleh **Red Hat Enterprise Linux** dan **OpenShift**. | Tidak direkomendasikan di RHEL / OpenShift. |
| **Kemudahan Instalasi** | Sudah preinstalled di RHEL/Fedora/CentOS Stream versi baru. | Harus diinstal manual (biasanya via `get.docker.com`). |
| **Daemon Process** | ❌ Tidak ada daemon | ✅ Ada daemon |
| **Tujuan Utama** | Enterprise, RHEL, OpenShift, keamanan tinggi. | Developer umum, CI/CD, pengembangan aplikasi. |

nah berikut ini beberapa command tau perintah yang sama yang ada di podman dan docker

| Tujuan | **Docker** | **Podman** |
|--------|-------------|-------------|
| Jalankan container HTTPD | `docker run -d -p 8080:80 httpd` | `podman run -d -p 8080:80 httpd` |
| Lihat container aktif | `docker ps` | `podman ps` |
| Hentikan container | `docker stop webserver` | `podman stop webserver` |
| Build image | `docker build -t myimage .` | `podman build -t myimage .` |

## Deploy
Langsung saja masuk ke deploymentnya,pertama-tama mulai untuk menyalakan labnya terlebih dahulu

![logo4](/assets/images/redhat/Screenshot 2025-11-06 141726.png)

tunggu beberapa saat untuk si labnya berjalan,setelah berjalan langsung klik workstation di labnya dan tunggu pop up dari windows workstationnya muncul,lalu jika sudah muncul 
masuk ke activities di sebelah pojok kanan atas dan cari icon terminal,dikarenakan saya masuk ke program openshift di redhat saya terlebih dahulu memulai dengan **lab start**,
setelah itu saya run **podman pull** selayaknya **docker pull**

![logo5](/assets/images/redhat/Screenshot 2025-11-06 140237.png)

setelah proses dari pull image selesai run image httpd yang sudah di pull tadi,ohh saya memakai registry dari rhelnya jadi bukan dari docker dikarenakan podman memang sangat di dukung 
di lingkungan rhel

![logo7](/assets/images/redhat/Screenshot 2025-11-06 140329.png)

dan disini saya memakai flag **-p** untuk port forwarding dan flag **--rm** untuk delete otomatis si container saat sudah selesai digunakan,jika sudah running bisa langsung cek
ada dua cara bisa menggunakan **curl** dari terminal atau langsung dari web browser seperti **firefox** 

![logo6](/assets/images/redhat/Screenshot 2025-11-06 140346.png)

disini saya menggunakan web browser untuk mengecek apakah si httpd bisa di akses dari host,cukup klik tulisan activities di pojok kanan atas lalu
cari firefox karena browser default untuk linux itu firefox,setelah itu ketik di search bar **localhost:8080** karena saya membuka di port 8080 maka saya ketik 8080

![logo10](/assets/images/redhat/Screenshot 2025-11-06 140418.png)

jika tampilan menunjukan output seperti gambar di atas maka httpd bisa di akses,bukan cuman di cli saja saya akan test menggunakan desktop podman,
untuk nextnya bisa langsung menggunakan terminal tambahan atau kill container yang sedang berjalan,setelah itu di terminal ketik **podman-desktop**

![logo11](/assets/images/redhat/Screenshot 2025-11-06 140458.png)

tunggu beberapa saat jika ada output seperti gambar di atas maka proses berhasil,perbersar layar jika ada tampilan berupa options pilih yang ada kata podman atau simbol atau gambar podman
nah jika sudah maka akan di bawa ke sebuah menu diatas

![logo12](/assets/images/redhat/Screenshot 2025-11-06 140615.png)

nah jika sudah melihat ada tampilan untuk image-image yang sudah pernah di pull dari registry,klik image httpd lalu klik tombol start,lalu kamu akan di arahkan ke menu yang sama dengan gambar di atas,atur untuk local portnya dimana kamu akan mengakses si httpd lalu jika sudah klik start,

![logo13](/assets/images/redhat/Screenshot 2025-11-06 140753.png)

jika sudah maka tampilan akan seperti di gambar,untuk mengecek lakukan langkah yang sebelumnya.
