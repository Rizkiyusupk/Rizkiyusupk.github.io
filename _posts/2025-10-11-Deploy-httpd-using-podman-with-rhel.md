---
title: "Deploy HTTPD Container di RHEL Menggunakan Podman (OpenShift Training Edition)"
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

---

