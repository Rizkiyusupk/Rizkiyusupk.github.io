---
title: "Building an Event-Driven AWS Pipeline with LocalStack & Terraform"
date: 2026-08-12
tags: [Clouds,linux, devops,system,infrastructure,Terraform,terraform,Aws,localstack]
header:
  teaser: /assets/images/aws/INFRASTUCTURE.jpg
categories: [DevOps, Clouds,linux,Linux,Server,iac,infrastructure]
---
![oihve](/assets/images/aws/INFRASTUCTURE.jpg)


### Overview

Seperti biasa jika sudah selesai dengan docs sekarang bagian ke error pagenya,langsung saja masuk ke pembahasannya


### License activation failed! 

Ketika pertama kali running localstack as container menggunakan docker akan muncul error message seeprti diatas,itu wajar karena belum terdaftar di website localstack dan
belum menggunakan token auth user,karena kebijakan terbaru localstack yang mewajibkan running menggunakan token auth jadinya harus menggunakan token auth


### Solve

tinggal daftar lalu login menggunakan akun google juga bisa,lalu copy token auth dan paste di command docker run,atau tinggal copy saja command ini

```
 docker run -d --name localstack   -e LOCALSTACK_AUTH_TOKEN=TOKEN_AUTH  -e PERSISTENCE=1   -e LAMBDA_DOCKER_NETWORK=bridge   -e
LOCALSTACK_HOST=localhost.localstack.cloud   -p 4566:4566   -p 4510-4559:4510-4559   -v ~/localstack-data:/var/lib/localstack   -v /var/run/docker.sock:/var/run/docker.sock
localstack/localstack
```
