---
title: "Scaling the Cluster to Multi-Site and Enforcing RBAC in an Event-Driven Pipeline"
date: 2026-09-14
infra_used: "Hybrid Cloud-Native Infrastructure"
---

![apivhbs](/assets/images/case-hybrid-aws-infra/ChatGPT Image Sep 1, 2026, 10_25_30 PM.png)

### Overview 

Kali ini saya akan membuat sebuah page baru khusus untuk case atau kasus pada setiap infrastructure yang sudah saya bangun sebelumnya,jadi pada case page ini berisi 
projek saya yang mengotak-atik infrastructure yang sudah dibangun entah itu menambahkan beban,menambahkan cluster,atau melakukan horizontal scaling atau vertikal scaling,
di projek kali ini saya akan menambahkan 1 cluster lagi atau scale cluster yang sudah pernah yaitu "Hybrid Cloud-Native Infrastructure",dan bukan hanya menambah cluster tapi juga untuk bot telegram di tambahkan sesuai dengan cluster yang ditambah,**NOTE!!! BANYAK FILE-FILE CONFIG YANG SAMA DENGAN PROJEK INFRA YANG MENJADI DASAR,MAKA DARI
ITU DIHARAPKAN MEMBACA TERLEBIH DAHULU PROJEK Hybrid Cloud-Native Infrastructure** jika ingin baca [klik disini](https://rizkiyusupk.github.io/devops/clouds/linux/server/iac/infrastructure/aws-2/),
langsung saja masuk ke pembahasannya

### Tools
Untuk Tools Masih sama karena ini menggunakan infrastructure yang sudah dibuat sebelumnya jadinya tidak ada perubahan dalam penggunaan tools 

### Setup 

Sampai pada bagian setup untuk langkah pertama-tama karena dalam case ini akan menambahkan cluster 2 hal yang paling pertama yaitu membuat image untuk cluster baru,karena 
menggunakan kvm/qemu dan perlu untuk membuat image jadinya langkah pertama dalam case kali ini buat image baru dengan nama node-cluster-2,misal "k8s-worker1-cluster-2"
dan di kvm/qemu tidak bisa menggunakan image yang sudah ada dan hanya akan menimbulkan error dan saling tabrakan dengan node yang menggunakan image cluster 1,tapi untuk 
pool dan volume utama tidak perlu di gandakan,langsung saja lihat masuk ke config

```
vim prep-cluster-2.tf
|
resource "libvirt_volume" "vm-master-cluster-2" {
  name   = "vm-master-cluster-2.qcow2"
  pool   = libvirt_pool.k8s_pool.name
  target = {
    format = {
      type = "qcow2"
    }
  }


  capacity = 10737418240

  backing_store = {
    path   = libvirt_volume.ubuntu_base.path
    format = {
      type = "qcow2"
    }
  }
}

resource "libvirt_cloudinit_disk" "vm-master-cluster-2" {
  name = "vm-master-cluster-2-spec"

  user_data = <<-EOF
  #cloud-config
  users:
    - name: ubuntu
      sudo: ALL=(ALL) NOPASSWD:ALL
      lock_passwd: false
      shell: /bin/bash
  chpasswd:
    list: |
      ubuntu:iki123
    expire: false
  ssh_pwauth: true
  packages:
    - openssh-server
  timezone: UTC
  EOF

 meta_data = <<-EOF
    instance-id: vm-master-cluster-2
    local-hostname: master-cluster-2
  EOF

  network_config = <<-EOF
    version: 2
    ethernets:
      interfaces:
        match:
          name: enp1s0
        dhcp4: true
  EOF
}

resource "libvirt_volume" "vm-master-cluster-2-cloudinit" {
  name = "vm-master-cluster-2-cloudinit.iso"
  pool = libvirt_pool.k8s_pool.name

  create = {
    content = {
      url = libvirt_cloudinit_disk.vm-master-cluster-2.path
    }
  }
}

resource "libvirt_volume" "vm-worker1-cluster-2" {
  name   = "vm-worker1-cluster-2.qcow2"
  pool   = libvirt_pool.k8s_pool.name
  target = {
    format = {
      type = "qcow2"
    }
  }


  capacity = 10737418240

  backing_store = {
    path   = libvirt_volume.ubuntu_base.path
    format = {
      type = "qcow2"
    }
  }
}



resource "libvirt_cloudinit_disk" "vm-worker1-cluster-2" {
  name = "vm-worker1-cluster-2-spec"

  user_data = <<-EOF
  #cloud-config
  users:
    - name: ubuntu
      sudo: ALL=(ALL) NOPASSWD:ALL
      lock_passwd: false
      shell: /bin/bash
  chpasswd:
    list: |
      ubuntu:iki123
    expire: false
  ssh_pwauth: true
  packages:
    - openssh-server
  timezone: UTC
 EOF

  meta_data = <<-EOF
    instance-id: vm-worker1-cluster-2
    local-hostname: worker-cluster-2
  EOF


  network_config = <<-EOF
    version: 2
    ethernets:
      interfaces:
        match:
          name: enp1s0
        dhcp4: true
  EOF
}

resource "libvirt_volume" "vm-worker1-cluster-2-cloudinit" {
  name = "vm-worker1-cloudinit-cluster-2.iso"
  pool = libvirt_pool.k8s_pool.name


  create = {
    content = {
      url = libvirt_cloudinit_disk.vm-worker1-cluster-2.path
    }
  }
}

resource "libvirt_volume" "vm-worker2-cluster-2" {
  name   = "vm-worker2-cluster-2.qcow2"
  pool   = libvirt_pool.k8s_pool.name
  target = {
    format = {
      type = "qcow2"
    }
  }


  capacity = 10737418240

  backing_store = {
    path   = libvirt_volume.ubuntu_base.path
    format = {
      type = "qcow2"
    }
  }
}

resource "libvirt_cloudinit_disk" "vm-worker2-cluster-2" {
  name = "vm-worker2-cluster-2spec"

  user_data = <<-EOF
  #cloud-config
  users:
    - name: ubuntu
      sudo: ALL=(ALL) NOPASSWD:ALL
      lock_passwd: false
      shell: /bin/bash
  chpasswd:
    list: |
      ubuntu:iki123
    expire: false
  ssh_pwauth: true
  packages:
    - openssh-server
  timezone: UTC
 EOF

  meta_data = <<-EOF
    instance-id: vm-worker2-cluster-2
    local-hostname: worker2-cluster-2
  EOF


  network_config = <<-EOF
    version: 2
    ethernets:
      interfaces:
        match:
          name: enp1s0
        dhcp4: true
  EOF
}

resource "libvirt_volume" "vm-worker2-cluster-2-cloudinit" {
  name = "vm-worker2-cloudinit-cluster-2.iso"
  pool = libvirt_pool.k8s_pool.name


  create = {
    content = {
      url = libvirt_cloudinit_disk.vm-worker2-cluster-2.path
    }
  }
}
```

diatas adalah contoh dari config pertama dari case ini,dan sebenarnya ini hanya copy paste dari projek Hybrid "Cloud-Native" Infrastructure jadinya hanya copy file prepnya
lalu paste dan ganti nama dengan nama node-cluster-2,tapi hanya config image yang di perlukan untuk bagian awal seperti pembuatan pool dan volume yang akan menyimpan image
tetap sama,jadinya hanya membuat image tanpa perlu membuat pool dan volume lagi.Masuk ke tahap selanjutnya yaitu computer

```
vim compute-cluster-2.tf
|
resource "libvirt_domain" "master-cluster-2" {
  name   = "k8s-master-clsuter-2"
  memory = 2097152
  vcpu   = 2
  type   = "kvm"

  os = {
    type         = "hvm"
    type_arch    = "x86_64"
    type_machine = "q35"
  }

  devices = {
    disks = [
      {
        source = {
          volume = {
            pool   = libvirt_pool.k8s_pool.name
            volume = libvirt_volume.vm-master-cluster-2.name
          }
        }
        target = {
          bus = "virtio"
          dev = "vda"
        }
        driver = {
          type = "qcow2"
        }
      },
      {
        device = "cdrom"
        source = {
          volume = {
            pool   = libvirt_pool.k8s_pool.name
            volume = libvirt_volume.vm-master-cluster-2-cloudinit.name
          }
        }
        target = {
          bus = "sata"
          dev = "sda"
        }
      }
    ]

     interfaces = [
      {
        type  = "network"
        model = { type = "virtio" }
        source = {
          network = {
            network = "default"
          }
        }
      }
    ]
  }

  running = true
}

resource "libvirt_domain" "worker1-cluster-2" {
  name   = "k8s-worker1-cluster-2"
  memory = 2097152
  vcpu   = 2
  type   = "kvm"

  os = {
    type         = "hvm"
    type_arch    = "x86_64"
    type_machine = "q35"
  }

  devices = {
    disks = [
      {
        source = {
          volume = {
            pool   = libvirt_pool.k8s_pool.name
            volume = libvirt_volume.vm-worker1-cluster-2.name
          }
        }
        target = {
          bus = "virtio"
          dev = "vda"
        }
        driver = {
          type = "qcow2"
        }
      },
      {
        device = "cdrom"
        source = {
          volume = {
            pool   = libvirt_pool.k8s_pool.name
            volume = libvirt_volume.vm-worker1-cluster-2-cloudinit.name
          }
        }
        target = {
          bus = "sata"
          dev = "sda"
        }
      }
    ]

     interfaces = [
      {
        type  = "network"
        model = { type = "virtio" }
        source = {
          network = {
            network = "default"
          }
        }
      }
    ]
  }

  running = true
}

resource "libvirt_domain" "worker2-cluster-2" {
  name   = "k8s-worker2-cluster-2"
  memory = 2097152
  vcpu   = 2
  type   = "kvm"

  os = {
    type         = "hvm"
    type_arch    = "x86_64"
    type_machine = "q35"
  }

  devices = {
    disks = [
      {
        source = {
          volume = {
            pool   = libvirt_pool.k8s_pool.name
            volume = libvirt_volume.vm-worker2-cluster-2.name
          }
        }
        target = {
          bus = "virtio"
          dev = "vda"
        }
        driver = {
          type = "qcow2"
        }
      },
      {
        device = "cdrom"
        source = {
          volume = {
            pool   = libvirt_pool.k8s_pool.name
            volume = libvirt_volume.vm-worker2-cluster-2-cloudinit.name
          }
        }
        target = {
          bus = "sata"
          dev = "sda"
        }
      }
    ]

     interfaces = [
      {
        type  = "network"
        model = { type = "virtio" }
        source = {
          network = {
            network = "default"
          }
        }
      }
    ]
  }

  running = true
}
```

masuk ke config kedua yaitu untuk compute atau pembuatan dari node atau domain jika dalam kvm/qemu.tentu saja sama memang cuman copy paste karena hanya membuat node lagi
jadinya cuman copy paste config yang sudah ada dan mengganti namanya dengan node-cluster-2,oke jika sudah sekarang coba untuk melakukan apply ohh sebelumnya untuk main.tf
atau init masih sama dengan projek infrastructurenya jadi masih mengikuti file main.tfnya,langsung saja

```
terraform apply
```

