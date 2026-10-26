---
title: "Pull Based Infrastructure Hub & Spoke "
date: 2026-10-22
infra_used: "Pull Based Argocd"
---

![asdjadvpisfb](/assets/images/hub_and_spoke/hb.png)

### Overview

Kali ini saya akan membuat sebuah case page dimana saya akan mengotak otak sebuah infrastructure yang sudah saya buat sebelumnya,nah kali ini saya 
akan mengotak atik infra dari pull based yang sudah saya buat dan sedikit mengubahnya,jika di projek utama infra pull based semua cluster memiliki 
satu argocd bisa di sebut bahwa infra itu stand alone argocd nah tapi untuk yang kali ini hanya akan ada satu argocd di dua cluster dan infra ini
disebut dengan hub and spoke infrastructure,dan untuk hub akan di cluster jakarta dan untuk spokenya akan di cluster bandung,oke tidak usah berlama-
lama langsung saja masuk ke pembahasannya

| Node        | CPU     | RAM  | Storage | Network                                             |
|-------------|---------|------|---------|---------------------------------------------------- |
| **Master-cluster-Jakarta**  | 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 1-cluster-Jakarta**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 2-cluster-Jakarta**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Master-cluster-Bandung**  | 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 1-cluster-Bandung**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Worker 2-Cluster-Bandung**| 2 cores | 2GB  | 10GB    | 1 Adapters   ( Static Ip )          |
| **Jenkins**                 | 7 cores | 7GB  | 240GB   |                Wlan                 |

### Structure Folder

Untuk structure folder akan terdiri dari dua folder yang pertama untuk terraform 

```
terraform-setup/
├── .terraform/
├── compute.tf
├── main.tf
├── prep-vm.tf
├── prep-2.tf
├── compute-cluster-2.tf
```

dan yang kedua yaitu ansible

```
k8s/
├── ansible.cfg
├── inventory
├── playbook-allow-port.yaml
├── playbook-ip.yaml
├── playbook-join.yaml
├── playbook-kubernetes.yaml
├── playbook-pkg.yaml
├── playbook-swap.yaml
├── playbook-config.yaml
├── argocd.yaml
```

### Reasoning
Kenapa saya memutuskan untuk membuat hub and spoke infra?? karena saya melihat bahwa infra stand alone itu tidak tersentralisasi atau terpusat 
pada satu cluster dan kurangnya aspek keamanan atau security aspect,dan juga gampang untuk di scale up jika saja ada cluster baru 

### Setup 
Oke untuk langkah pertama cek apakah masih ada application di dalam argocd  di cluster bandung,jika ada hapus terlebih dahulu,lalu untuk langkah selanjutnya bisa di pilih 
apakah mau di uninstall atau tidak untuk argocd cluster bandung,meskipun argocd cluster bandung masih ada dan tidak di hapus tidak apa apa tidak berpengaruh,karena
application yang ada di dashboard ui argocd bandung sudah di hapus,jadinya hanya akan ada satu argocd yang akan mengarah ke app config bandung yang di projek infra awal
sebelumnya di buat,oke jadi yang harus di pastikan di tahap ini adalah tidak ada application di dashboard argocd bandung,oke jika sudah masuk ketahap selanjutnya yaitu
merging kubeconfig files,yaitu agar si cluster jakarta punya 2 context dalam satu cluster,pertama tama buat tunnling dari argocd server ke dalam localhost gunakan command

```
kubectl port-forward svc/argocd-server open-port:80 --address=0.0.0.0 -n argocd &
```

selanjutnya saya ingin menjelaskan terlebih dahulu apa itu context dalam k8s,jadi agar kamu bisa mengoperasikan kubernetes lewat cli kamu perlu yang namanya kubectl,nah 
kubectl atau kubecontrol ini lah yang dipakai oleh kamu nantinya kubectl bicara lewat kubeconfig yang sudah tersimpan di 

```
$HOME/user/.kube/config
```

nah kubconfig itu tersimpan disana di dalam kubeconfig itu salahs atunya ialah context biasanya seperti ini

```
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
```

nah kurang lebih isinya context seperti diatas ada cluster,user,nama dari context kubernetes,lalu ada current context sama dengan nama context,lalu yang terakhir ada nama 
untuk user atau client yang biasanya nanti di ujung kubconfig ada certnya sendiri,nah perlu dicatat bahwa ketika akan merging dengan kubeconfig lainnya yaitu perlu 
ganti nama dan sesesuaikan dengan nama cluster,oke masuk ke step kembali cat kubeconfig cluster bandung lalu copy isinya jika sudah masuk ke terminal di cluster jakarta,
lalu buat copy kubeconfig cluster jakarta

```
cp .kube/config .kube/config.bak-jakarta
```

oke jika sudah buat file untuk kubeconfig dari cluster bandung

```
vim .kube/config-bandung
```

lalu paste semua isi dari copy kubeconfig bandung,jika sudah edit bagian context tadi 

```
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin <- ganti
  name: kubernetes-admin@kubernetes <- ganti
current-context: kubernetes-admin@kubernetes <- ganti
kind: Config
preferences: {}
users:
- name: kubernetes-admin <- ganti
  user:
```

tujuan dari mengganti ini supaya tidak terjadi tabrakan dengan cluster jakarta dan kubernetes tidak kebingungan dengan dua context yang sama,lalu jika sudah masuk ke tahap
merging selanjutnya yaitu tahap flatten,tahap dimana kubeconfig bandung akan di ratakan lalu di satukan dengan kubeconfig

```
KUBECONFIG=$HOME/.kube/config:$HOME/.kube/config-surabaya kubectl config view --flatten > /tmp/merged
mv /tmp/merged ~/.kube/config && chmod 600 ~/.kube/config
```

oke jika sudah sekarang cek untuk isi dari file config di cluster jakarta 
sharusnya ada dua context,jika sudah ada dua context selanjutnya login session agar nanitnya argocd bisa tau bahwa ada cluster lain yang ingin bergabung dengan nah masih
ingat tadi sudah melakukan kubectl port forward fungsi dari port forward itu ialah untuk login session ini agar argocd bisa menambahkan kubeconfig cluster bandung ke 
cluster jakarta

```
aargocd login localhost:localport --username admin --password default-password --insecure
```

nah untuk default password sudah di bicarakan sebelumnya jadinya harus tau password buat masuk ke dashboard argocd itu loh,lalu untuk localport itu adalah port dimana
kubectl port forward tadi di buka,oke jika sudah login gunakan command tinggal add deh gunakan command

```
argocd cluster add name of your context --name name
```

ganti jadi nama context mu lalu nama buat context mu itu jika sudah tunggu hingga prosesnya selesai,jika sudah selesai sekarang cek dengan menggunakan command

```
argocd cluster list
```

maka seharusnya akan ada dua cluster yang muncul 

![asdkaudvs](/assets/images/hub_and_spoke/Screenshot 2026-10-22 234037.png)

oke selamat sudah berhasil menggabungkan cluster bandung ke cluster jakarta jadinya jakarta sebagai hub dan bandung sebagi spokee,test apakah benar-benar ada di dashboard
argocd,masuk ke argocd lalu klik create application setelah itu scroll ke bawah sampai ke bagian destination

![ascsdubc](/assets/images/hub_and_spoke/Screenshot 2026-10-26 205532.png)

klik bagian kosong di bar destination akan ada section yang muncul dan menampilkan cluster apa saja yang ada jika tahap tadi benar maka seharusnya akan muncul ip dari 
cluster badung seperti diatas lalu coba ganti ke name dari url seharusny akan menjadi seperti ini

![as8awodh](/assets/images/hub_and_spoke/Screenshot 2026-10-26 205545.png)

**NOTE ABAIKAN CLUSTER SURABAYA!!**
nah untuk bagian name ini sebenarnya sama saja cuman bedanya context yang diambil itu adalah name dari cluster yang tertulis di config yang ini loh

```
argocd cluster add name of your context --name name <--- tuh berdasarkan name ini
```

jadi bisa di sesuaikan dengan nama yang di inputkan cancel pembuatan application,oke karena sudah melakukan penggabungan config dari kedua cluster masuk ke tahapan 
selanjutnya yaitu pembuatan projects nah kenapa sih project itu penting?? itu karena project yang membuat sebuah deployment tetap terdeploy di mana seharusnya jadinya
misalkan ada apps1-jakarta-frontend dan apps2-jakarta-backend lalu ada app2-jakarta-frontend dan apps2-jakarta-backend nah itukan harus masing masing memang sama sama
cluster site jakarta tapi untuk apps beda project makanya projects itu penting untuk isolasi tambahan agar ketika melakukan deployment semua deployment terdeploy dimana
seharusnya jadinya tdak bisa sembarangan terdeploy,oke masuk langsung ke pembuatannya pergi ke settings di bagian kiri 

![asdoijs](/assets/images/hub_and_spoke/Screenshot 2026-10-26 211133.png)

masuk ke section project lalu akan di bawa ke menu seperti ini 

![asdubac](/assets/images/hub_and_spoke/Screenshot 2026-10-26 211836.png)

lalu buat sebuah project dengan nama bebas sesuai dengan setelah itu akan 
dan arahkan endpoint dari projects ke repo config yang di miliki oleh repo jakarta,
