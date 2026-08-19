---
title: "Debugging Building an Event-Driven AWS Pipeline with LocalStack & Terraform"
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

### awslocal: command not found

error ini terjadi ketika selesai melakukan installasi aws wrapper,karena aws wrapper terinstall tidak di direktori bin sistem maka ketika user ingin test menggunakan aws 
wrapper,jadinya sistem kebingungan ketika user mengetik awslocal padahal program malah terinstall di dalam direktori bin user bukan sistem


### Solve

karena penyebab erorr sudah di temukan jadi langsung eksekusi saja pertama-tama cek apakah benar aws wrapper terinstall di direkori bin user gunakan comamnd

```
ls .local/bin | grep aws
```

jika sudah export path dari .local/bin atau direktori bin user dimana aws wrapper terinstal,jika sudah expprt ke bashrc

```
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

command diatas akan memastikan bahwa export path permanen menempel ke bashrc dan sistem akan tahu dimana aws wrapper terinstall


### Aws not installed

oke error ini terjadi ketika saya lupa dengan aws cli,saya pikir saya sudah install aws cli rupanya aws wrappe saja yang sudah saya install dan ya akan keluar error 
message seperti ini

```
[Errno 2] No such file or directory: b'/snap/bin/aws'
```

### Solve

karena akar masalah sudah ditemukan langsung saja ke intinya jadi tinggall install saja aws cli nya

```
pip install awscli --break-system-packages
```

### Aws try to connect to real aws account

masuk ke error selanjutnya yaitu ketika init pertama kali file aws.tf yang berfungsi sebagai jembatan bagi terraform dan localstack malah mencoba untuk connect ke akun aws
asli karena file aws.tf saya sebelumnya itu tidak menggunaakn 

```
  s3_use_path_style           = true
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  skip_requesting_account_id  = true
```

jadinya akan ada error message seperti ini

```
│ Error: Retrieving AWS account details: validating provider credentials: retrieving caller identity from STS: operation error STS: GetCallerIdentity, https response error
StatusCode: 403, RequestID: 542cd878-b033-419b-a5d6-9a0b21749529, api error InvalidClientTokenId: The security token included in the request is invalid.
│
│   with provider["registry.terraform.io/hashicorp/aws"],
│   on aws.tf line 1, in provider "aws":
│    1: provider "aws" {
│
╵
```


### Solve 

karena error sudah di temukan tinggal solve saja dengan menambahkan baris baris diatas,untuk penjelasan baris apa saja itu ada di docs

```
provider "aws" {
  access_key                  = "test"
  secret_key                  = "test"
  region                      = "us-east-1"

  s3_use_path_style           = true
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  skip_requesting_account_id  = true

  endpoints {
    s3  = "http://localhost:4566"
    iam = "http://localhost:4566"
    sts = "http://localhost:4566"
    sqs = "http://localhost:4566"
    sns = "http://localhost:4566"
    dynamodb = "http://localhost:4566"
    lambda = "http://localhost:4566"
    logs = "http://localhost:4566"
    cloudwatch = "http://localhost:4566"
  }
}
```

semuanya sudah di jelaskan di docs 


