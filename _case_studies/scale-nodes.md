---
title: "Scaling the Cluster to Multi-Site and Enforcing RBAC in an Event-Driven Pipeline"
date: 2026-09-14
infra_used: "Hybrid Cloud-Native Infrastructure"
---

![apivhbs](/assets/images/case-hybrid-aws-infra/ChatGPT Image Sep 1, 2026, 10_25_30 PM.png)

### Overview 

Kali ini saya akan membuat sebuah page baru khusus untuk case atau kasus pada setiap infrastructure yang sudah saya bangun sebelumnya,jadi pada case page ini berisi 
projek saya yang mengotak-atik infrastructure yang sudah dibangun entah itu menambahkan beban,menambahkan cluster,atau melakukan horizontal scaling atau vertikal scaling,
di projek kali ini saya akan menambahkan 1 cluster lagi atau scale cluster yang sudah pernah yaitu "Hybrid Cloud-Native Infrastructure",dan bukan hanya menambah cluster 
tapi juga untuk bot telegram di tambahkan sesuai dengan cluster yang ditambah,**NOTE!!! BANYAK FILE-FILE CONFIG YANG SAMA DENGAN PROJEK INFRA YANG MENJADI DASAR DAN SEMUA 
KONDISI HARUS BENAR-BENAR SAMA ENTAH ITU SSH ATAU SETIAP CONFIG,MAKA DARI ITU DIHARAPKAN MEMBACA TERLEBIH DAHULU PROJEK Hybrid Cloud-Native Infrastructure** jika ingin 
baca [klik disini](https://rizkiyusupk.github.io/devops/clouds/linux/server/iac/infrastructure/aws-2/),langsung saja masuk ke pembahasannya

### Tools
Untuk Tools Masih sama karena ini menggunakan infrastructure yang sudah dibuat sebelumnya jadinya tidak ada perubahan dalam penggunaan tools 

### Reasoning
Kenapa saya memilih untuk menambahkan cluster? Awalnya saya hanya ingin menerapkan konsep RBAC ke infrastructure yang sudah dibangun. Tapi saya berpikir, untuk 1 cluster 
saja, penerapan RBAC-nya tidak terlalu menantang — sebatas membuat manifest dan apply ke satu API server. Karena itu saya putuskan untuk sekaligus menambahkan cluster 
baru, supaya saya juga bisa membuktikan pemahaman bahwa RBAC itu scoped per-cluster — tidak ada RBAC cross-cluster bawaan Kubernetes, jadi config harus dipisah dan 
diterapkan independen ke masing-masing cluster. Saya pakai blueprint infrastructure yang sama untuk kedua cluster, supaya perbandingan penerapan RBAC-nya konsisten dan 
environment-nya apple-to-apple.

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

oke jika sudah berhasil cek menggunakan command

```
virsh list --all
```

![asdvdovsv](/assets/images/case-hybrid-aws-infra/Screenshot 2026-09-14 221707.png)

jika sudah output yang diharapkan akan seperti output diatas ada 6 node yang berjalan masing-masing di bagi 3 jadi ada 2 cluster,saya beri nama cluster 2 itu cluster 
bandung dan untuk cluster 1 cluster jakarta,oke jika sudah maka selamat node sudah berhasil berjalan selanjutnya yaitu bagian installasi k8s di node cluster bandung
gampang saja tinggal jalankan ansible playbook yang sama dan hanya menambahkan ip baru ke inventory 

```
vim inventory
|
[Jenkins]
rizky ansible_host=192.168.100.7 ansible_user=rizky ansible_password=iki123

[masters]
k8s-master ansible_host=10.10.10.80 ansible_user=ubuntu ansible_password=iki123

[workers]
k8s-worker1 ansible_host=10.10.10.34 ansible_user=ubuntu ansible_password=iki123
k8s-worker2 ansible_host=10.10.10.35 ansible_user=ubuntu ansible_password=iki123


[masters-cluster-2]
k8s-master-cluster-2 ansible_host=10.10.10.222 ansible_user=ubuntu ansible_password=iki123

[workers-cluster-2]
k8s-worker1-cluster-2 ansible_host=10.10.10.123 ansible_user=ubuntu ansible_password=iki123
k8s-worker2-cluster-2 ansible_host=10.10.10.67 ansible_user=ubuntu ansible_password=iki123
```

oke jika sudah tinggal tambahkan saja di hosts di bagian atas playbook,semua confignya sama tinggal tambahkan hostnya lalu run,jika sudah install cni untuk network k8s

```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

lalu simpan kubeconfignya

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

oke sekarang test apakah berhasil 

```
kubectl get nodes
```

![adovn](/assets/images/case-hybrid-aws-infra/Screenshot 2026-09-14 230049.png)

oke sudah berjalan clusternya,lanjut ke tahap berikutnya yaitu pembuatan bot tele,tidak usah berlama-lama lagi caranya masih sama seperti di projke Hybrid Aws
Infrastrcuture,baca terlebih dahulu lalu balik lagi kesini,okee jika sudah mendapatkan bot tokennya saya harap ada dua bot token tambahan jadi nantinya ada 4 bot yang aktif
2 bot alert dan 2 bot aws lambda kenapa saya menggunakan 4 bot? karena memang perlu untuk mengtahui secara detail dan eksplisit tentang log,data,trace dari setiap 
aktifitas atau node yang berjalan jadinya sangat penting jika ingin mementingkan aspek detail dan jika saja ada 1 bot dan bot itu terkena masalah seperti rate limit atau 
apapun yang bisa menyebabkan workflow terhenti karena 1 bot bermasalah saya masih punya yang bot lainnya,jangan lupa edit lagi file terraform.tfvarsnya 
masukan bot token ke filenya

```
telegram_bot_token_site_jakarta = "YOUR_BOT_TOKEN"
telegram_chat_id_jakarta   = "1854226173"

telegram_bot_token_site_bandung = "YOUR_BOT_TOKEN"
telegram_chat_id_bandung   = "1854226173"
```

setelah itu langsung saja edit untuk bagian lambda consumer

```
vim lambda_function_consumer.py
|
import json
import time
import os
import urllib.request
import urllib.error
import boto3

dynamodb = boto3.resource('dynamodb')
TABLE_NAME = "PipelineHistory"
table = dynamodb.Table(TABLE_NAME)


BOT_CONFIG = {
    "jakarta": {
        "token": os.environ.get("TELEGRAM_BOT_TOKEN_JAKARTA"),
        "chat_id": os.environ.get("TELEGRAM_CHAT_ID_JAKARTA"),
    },
    "bandung": {
        "token": os.environ.get("TELEGRAM_BOT_TOKEN_BANDUNG"),
        "chat_id": os.environ.get("TELEGRAM_CHAT_ID_BANDUNG"),
    },
}


def detect_site(body, original_message):
    try:
        attrs = body.get('MessageAttributes', {})
        site_attr = attrs.get('site', {}).get('Value')
        if site_attr:
            return site_attr.lower()
    except Exception:
        pass

    text_lower = original_message.lower()
    if "bandung" in text_lower:
        return "bandung"
    if "jakarta" in text_lower:
        return "jakarta"

    return "jakarta"


def send_telegram_alert(site, text):
    config = BOT_CONFIG.get(site)
    if not config:
        print(f"Site '{site}' tidak dikenal, skip alert")
        return

    token = config["token"]
    chat_id = config["chat_id"]

    if not token or not chat_id:
        print(f"Bot token/chat_id untuk site '{site}' belum diset, skip alert")
        return

    url = f"https://api.telegram.org/bot{token}/sendMessage"
    payload = json.dumps({
        "chat_id": chat_id,
        "text": f"[{site.upper()}] {text}"
    }).encode('utf-8')

    req = urllib.request.Request(
        url,
        data=payload,
        headers={"Content-Type": "application/json"},
        method="POST"
    )
    try:
        with urllib.request.urlopen(req, timeout=8) as response:
            print(f"Telegram alert ({site}) terkirim, status: {response.status}")
    except urllib.error.HTTPError as e:
        print(f"Telegram API error ({site}): {e.code} - {e.read().decode('utf-8')}")
    except urllib.error.URLError as e:
        print(f"Gagal konek ke Telegram API ({site}): {str(e)}")


def lambda_handler(event, context):
    print(f"RAW EVENT: {json.dumps(event)}")

    for record in event['Records']:
        body = json.loads(record['body'])
        original_message = body.get('Message', 'Pesan tidak ditemukan')
        sns_message_id = body.get('MessageId', record.get('messageId', 'unknown'))

        site = detect_site(body, original_message)
        print(f"Pesan diterima dari SQS (site: {site}): {original_message}")

        item = {
            'event_id': sns_message_id,
            'timestamp': int(time.time() * 1000),
            'message': original_message,
            'source': 'sqs-consumer-lambda',
            'site': site,
            'status': 'processed',
        }

        try:
            table.put_item(Item=item)
            print(f"Berhasil ditulis ke DynamoDB: {item['event_id']}")
        except Exception as e:
            print(f"GAGAL nulis ke DynamoDB: {str(e)}")
            send_telegram_alert(site, f"GAGAL nulis history ke DynamoDB!\n{original_message}\nError: {str(e)}")
            raise

        send_telegram_alert(site, f"Pipeline event diproses:\n{original_message}")

    return {
        'statusCode': 200,
        'body': json.dumps('Pesan berhasil diproses, dicatat ke DynamoDB, dan alert terkirim')
    }
```

oke kode lambda diatas menggunakan python dan akan secara otomatis memfilter path /jakarta atau /bandung tergantung kata apa yang nantinya digunakan oleh user,misalkan 
di jenkinsfile di tulis begini

```
awslocal s3 cp kube.log s3://my-bucket/AWSLogs/jakarta/kube-site-jakarta.log
```

yang menjadi filter di contoh itu ya kata /jakarta atau /bandung dan jika tidak ada dua kata itu tenang di dalam code ada fallback otomatis yang langsung masuk ke /jakarta
bot,saya di bantu claude untuk membbuat lambda codenya hehe :v,oke lanjut edit juga file lambda-2.tf atau config lambdanya

```
lambda-2.tf
|
resource "aws_lambda_function" "lambda-function-consumer" {
  filename          = "${path.module}/lambda_function_consumer.zip"
  function_name     = "lambda_function_consumer"
  role              = aws_iam_role.lambda-consumer-role.arn
  handler           = "lambda_function_consumer.lambda_handler"
  source_code_hash  = filebase64sha256("${path.module}/lambda_function_consumer.zip")
  runtime           = "python3.12"
  timeout           = 10

  environment {
    variables = {
      TELEGRAM_BOT_TOKEN_JAKARTA = var.telegram_bot_token_site_jakarta
      TELEGRAM_CHAT_ID_JAKARTA   = var.telegram_chat_id_jakarta
      TELEGRAM_BOT_TOKEN_BANDUNG = var.telegram_bot_token_site_bandung
      TELEGRAM_CHAT_ID_BANDUNG   = var.telegram_chat_id_bandung
    }
  }
  tags = {
    Environment = "production"
    Application = "example"
  }
}

variable "telegram_bot_token_site_jakarta" {
  type      = string
  sensitive = true
}

variable "telegram_chat_id_jakarta" {
  type      = string
  sensitive = true
}

variable "telegram_bot_token_site_bandung" {
  type      = string
  sensitive = true
}

variable "telegram_chat_id_bandung" {
  type      = string
  sensitive = true
}
```

code diatas hanya menambahkan dua variable lagi dan sedikit mengubah nama dari var yang tadinya hanya 

```
TELEGRAM_BOT_TOKEN  = var.telegram_bot_token
TELEGRAM_CHAT_ID    = var.telegram_chat_id
```

diubah menjadi 

```
TELEGRAM_BOT_TOKEN_JAKARTA = var.telegram_bot_token_site_jakarta
TELEGRAM_CHAT_ID_JAKARTA   = var.telegram_chat_id_jakarta
TELEGRAM_BOT_TOKEN_BANDUNG = var.telegram_bot_token_site_bandung
TELEGRAM_CHAT_ID_BANDUNG   = var.telegram_chat_id_bandung
```

tidak ada banyak perubahan hanya ada penambahan nama seperti "TOKEN_SITE" dan "CHAT_ID_SITE" kurang lebih sama seperti yang ada di projek infra karena ya memang cuman 
menambahkan saja hehhehhe,oke jika sudah pastikan hapus file zip lama dari lambda_function_consumer lalu zip ulang

```
rm lambda_function_consumer.zip
zip lambda_function_consumer.zip lambda_function_consumer.py
```

oke jika sudah tinggal terraform apply lagi,**NOTE KONDISI DISAAT INI SAYA HARAP SEMUA ENVIRONMENT BERJALAN SESUAI DENGAN KONDISI DI PROJEK INFRASTRUCTURE JADINYA TINGGAL
RUNNING LAGI ATAU APPLY LAGI SAJA TAPI JIKA BELUM DISAAT IN DOCKER HARUS SUDAH BERJALAN KEMBALI KE PROJEK INFRASTRUCTURE UNTUK MELIHAT BAGAIMANA CARA RUNNING DOCKER 
LOCALSTACK**

```
terraform apply
```

oke jika sudah selesai saatnya test manual untuk test manualnya cukup dengan menggunakan command

```
awslocal s3 cp kube.log s3://my-tf-test-bucket/AWSLogs/bandung/kube-bandung.log
```

oke tunggu hingga selesai jika command berhasil terkeskusi cek secara berkalan notifikasi di handphone akan ada pesan dari telegram

![iphrdrb](/assets/images/case-hybrid-aws-infra/1789481633672.jpg)

oke karena sudah ada notifikasi dari telegram artinya workflownya berjalan dengan baik,oke masuk ke tahap selanjutnya yaitu rbac,untuk bagian rbacnya dimulai dengan 
pembuatan namespace dari masing-masing cluster 

```
kubectl create namespace app-site
|
kubectl create namespcae app-jakarta (cluster jakarta)
kubectl create namespace app-bandung (cluster bandung)
```

jika sudah masuk ke bagian config pertama yaitu config service account

```
vim service-account.yaml
|
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-deployer
  namespace: app-$site
```

untuk bagian config service account dengan nama jenkins-deployer dan dengan namespace masing-masing cluster jadi misalkan untuk cluster jakarta tinggal ganti namanya saja
di ujung jadi 

```
namespace: app-jakarta
```

begitu pula dengan cluster bandung,oke jika sudah run file dengan menggunakan command

```
kubectl apply -f service-account.yaml
```

oke jika sudah masuk ke config selanjutnya yaitu pembuatan dari clusterrole,oke saya disini akan menggunakan clusterrole kenapa clusterrole? supaya scopednya bisa reach 
cluster level,kenapa engga role biasa karena role biasa tidak bisa reach scoped cluster jadinya hanya bisa menjangkan namespace saja,lalu untuk binding sedikit berbeda
biasanya jika menggunakan clusterrole maka untuk binding akan menggunakan clusterrole binding,kenapa saya menggunakan rolebinding? karena untu lebih mudah mengatuh 
scopednya jadinya untuk permission yang di berikan itu tidak namespace scoped tapi cluster scoped dan untuk bindingnya hanya namespace scoped,oke langsung saja tanpa

```
vim clusterrole.yaml
|
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-deployler-role
rules:
- apiGroups: ["","apps"]
  resources: ["secrets","services","pods","deployments"]
  verbs: ["get", "watch", "list","get", "watch", "create", "update", "patch", "delete"]
```

disini saua menggunakn rules untuk apps dan generals dengan memberikan beberapa resource sperti akses ke secrets,services,pods,dan deployments, saya tidak memberikan akses 
ke nodes karena sensitive,lalu untuk verbs ya standar saja seperti get,delete,update,create,dll jika sudh gunakan command aapply

```
kubectl apply -f clusterrole.yaml
```

lanjut ke rolebinding.yaml

```
vim role-binding.yaml
|
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins-role-binding
  namespace: app-jakarta
subjects:
- kind: ServiceAccount
  name: jenkins-deployer
  namespace: app-jakarta
roleRef:
  kind: ClusterRole
  name: jenkins-deployler-role
  apiGroup: rbac.authorization.k8s.io
```

tidak ada banyak hal yang akan saya beritahu soalnya saya sudah memberitahu di awal bahwa saya menggunakan role binding bukan clusterrole binding,jika sudah langsung
apply 

```
kubectl apply -f role-binding.yaml
```
