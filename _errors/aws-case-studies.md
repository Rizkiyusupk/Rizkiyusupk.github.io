---
title: "Debugging: Scaling the Cluster to Multi-Site and Enforcing RBAC in an Event-Driven Pipeline"
date: 2026-09-17
infra_used: "Hybrid Cloud-Native Infrastructure"
---

### Overview

Seperti biasa kali ini masuk ke error page dimana sesi ini membahas mengenai error error apa saja yang saya temukan ketika membangung projek [ini](https://rizkiyusupk.github.io/case-studies/scale-nodes/)

### Error Nodes

Error ini terjadi ketika pertam kali saya akan melakukan scaling node hal pertama yang saya lakukan ialah langsung melakukan terraform apply tanpa membuat image dan cloud init terpisah terlebih
dahulu padahal rulesnya sudah jelas bahwa tidak boleh berbagi image atau cloud init jadinya harus buat lagi

### Solve

karena sudah ditemukan penyebabnya tinggal solving yaitu dengan membuat image lagi dan jika mengikuti sesuai dengan post tidak akan ada error seperti ini


### Pipeline Wont Work

Error ini bisa terjadi karena pluggin di jenkins belum terinstall jadi mau bagaimanapun jenkins pipeline berjalan akan tetap error


### Solve

Untuk solve error ini cukup mudah tinggal install  pluginnya


### Lambda Always Fallback

Error ini terjadi karena saya ceroboh lupa menambahkan bagian 

```
/jakarta atau /bandung
```

di dalam command di jenkinsfile jadinya lambda akan selalu fallback ke jakarta bot

### Solve

Mudah saja tinggal tambahkan 

```
/jakarta atau /bandung
```

