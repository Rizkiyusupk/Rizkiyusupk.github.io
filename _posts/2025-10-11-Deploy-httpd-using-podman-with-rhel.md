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
tool yang populer untuk micro services atau container 
![logo3](/assets/images/redhat/applsci-12-06737-g001.png)
