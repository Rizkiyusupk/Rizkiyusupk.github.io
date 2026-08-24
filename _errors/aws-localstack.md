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

### Docker not running

Error ini terjadi ketika docker localstack tidak running atau berjalan tetapi sedang mengekskusi file tf jadinya akan ada error message sepert inini

```
aws_s3_bucket.test-bucket: Still creating... [00m43s elapsed]
aws_s3_bucket.test-bucket: Still creating... [00m53s elapsed]
aws_s3_bucket.test-bucket: Still creating... [01m06s elapsed]
aws_s3_bucket.test-bucket: Still creating... [01m16s elapsed]
aws_s3_bucket.test-bucket: Still creating... [01m26s elapsed]
aws_s3_bucket.test-bucket: Still creating... [01m39s elapsed]
aws_s3_bucket.test-bucket: Still creating... [01m49s elapsed]
aws_s3_bucket.test-bucket: Still creating... [01m59s elapsed]
aws_s3_bucket.test-bucket: Still creating... [02m12s elapsed]
aws_s3_bucket.test-bucket: Still creating... [02m22s elapsed]
aws_s3_bucket.test-bucket: Still creating... [02m32s elapsed]
aws_s3_bucket.test-bucket: Still creating... [02m45s elapsed]
aws_s3_bucket.test-bucket: Still creating... [02m55s elapsed]
aws_s3_bucket.test-bucket: Still creating... [03m05s elapsed]
aws_s3_bucket.test-bucket: Still creating... [03m15s elapsed]
aws_s3_bucket.test-bucket: Still creating... [03m28s elapsed]
aws_s3_bucket.test-bucket: Still creating... [03m38s elapsed]
aws_s3_bucket.test-bucket: Still creating... [03m48s elapsed]
aws_s3_bucket.test-bucket: Still creating... [04m01s elapsed]
aws_s3_bucket.test-bucket: Still creating... [04m11s elapsed]
aws_s3_bucket.test-bucket: Still creating... [04m21s elapsed]
aws_s3_bucket.test-bucket: Still creating... [04m34s elapsed]
aws_s3_bucket.test-bucket: Still creating... [04m44s elapsed]
aws_s3_bucket.test-bucket: Still creating... [04m54s elapsed]
aws_s3_bucket.test-bucket: Still creating... [05m07s elapsed]
aws_s3_bucket.test-bucket: Still creating... [05m17s elapsed]
aws_s3_bucket.test-bucket: Still creating... [05m27s elapsed]
aws_s3_bucket.test-bucket: Still creating... [05m40s elapsed]
aws_s3_bucket.test-bucket: Still creating... [05m50s elapsed]
aws_s3_bucket.test-bucket: Still creating... [06m00s elapsed]
aws_s3_bucket.test-bucket: Still creating... [06m10s elapsed]
aws_s3_bucket.test-bucket: Still creating... [06m23s elapsed]
aws_s3_bucket.test-bucket: Still creating... [06m33s elapsed]
aws_s3_bucket.test-bucket: Still creating... [06m43s elapsed]
aws_s3_bucket.test-bucket: Still creating... [06m56s elapsed]
aws_s3_bucket.test-bucket: Still creating... [07m06s elapsed]
aws_s3_bucket.test-bucket: Still creating... [07m16s elapsed]
^C
Interrupt received.
Please wait for Terraform to exit or data loss may occur.
Gracefully shutting down...

Stopping operation...
╷
│ Error: execution halted
│
│
╵
╷
│ Error: execution halted
│
│
╵
╷
│ Error: reading S3 Bucket (my-tf-test-bucket): empty result
│
│   with aws_s3_bucket.test-bucket,
│   on s3.tf line 1, in resource "aws_s3_bucket" "test-bucket":
│    1: resource "aws_s3_bucket" "test-bucket" {
│
╵
```

terraform akan stuck terus menerus didalam sebuah proses karena docker tidak berjalan atau container docker tidak berjalan


### Solve

tinggal jalankan saja docker container atau jika docker mati nyalakan docker


```
docker run -d --name localstack   -e LOCALSTACK_AUTH_TOKEN=TOKEN_AUTH   -e PERSISTENCE=1   -e LAMBDA_DOCKER_NETWORK=bridge   -e
LOCALSTACK_HOST=localhost.localstack.cloud   -p 4566:4566   -p 4510-4559:4510-4559   -v ~/localstack-data:/var/lib/localstack   -v
/var/run/docker.sock:/var/run/docker.sock   localstack/localstack
```

atau copy saja command diatas

### Old log 

ketika testing dengan melakukan beberapa test kadang ada beberapa log lama yang masih tersimpan,jadinya old baru tidak akan keluar dan didalam cloud watch group log
malah hanya ada log lama yang tersimpan


### Solve

Hapus terlebih dahulu log lama atau jika manghapus log lama masih tidak berhasil hancurkan atau delete docker container lalu run ulang lagi untuk testing akhir

```
awslocal logs delete-log-group --log-group-name 
```

atau 

```
docker stop id_container
docker rm id_container
```

lalu start lagi

```
docker run -d --name localstack   -e LOCALSTACK_AUTH_TOKEN=TOKEN_AUTH   -e PERSISTENCE=1   -e LAMBDA_DOCKER_NETWORK=bridge   -e
LOCALSTACK_HOST=localhost.localstack.cloud   -p 4566:4566   -p 4510-4559:4510-4559   -v ~/localstack-data:/var/lib/localstack   -v
/var/run/docker.sock:/var/run/docker.sock   localstack/localstack
```

### Lambda function don't work 

Error ini biasanya terjadi ketika menulis kode lambda trus lupa untuk menzipnya kembali,jika saja di file config menambahkan bagian untuk menzip sebuah file sesuai dengan
file lambda maka tidak akan ada error ini,kareana kode config tf saya itu tidak menambahakan bagian untuk zip file lambda didalam file config maka dari itu saya mendapatkan
error bahwa lambda tidak melakukan apapun,

### Solve

Untuk solve problem ini cukup sederhana hanya dengan melakkan zip setiap melakukan perubahan di file lambda


### Network issue 

Error ini terjadi karena lambda mencoba mencari dimana ip dari si container ketika kode dijalankan,lambda malah kebingungan mencari dimana network si container localstack 
berada,jadinya proses terrafom apply akan stuck dan mengulang-ulang secara terus menerus karena lambda kebingungan.output error akan seperti ini

```
time="2026-08-08T14:44:41Z" level=fatal msg="Failed to send status ready to LocalStack Post
\"http://172.17.0.2:4566/_localstack_lambda/ade38cf0a9c73bb9bf1ee2211f8beff5/status/ade38cf0a9c73bb9bf1ee2211f8beff5/ready\": dial tcp 172.17.0.2:4566: connect: connection
refused. Exiting." func=main.main file="/home/runner/work/lambda-runtime-init/lambda-runtime-init/cmd/localstack/main.go:387"
```

### Solve

karena masalahnya sudah ditemukan dan hal itu terjadi karena network issue maka solusi dari error ini cukup tambahkan variable untuk menentukan tipe network untuk memecah 
masalah ini

```
docker run -d --name localstack   -e LOCALSTACK_AUTH_TOKEN=TOKEN-AUTH   -e PERSISTENCE=1   -e LAMBDA_DOCKER_NETWORK=bridge   -e LOCALSTACK_HOST=localhost.localstack.cloud
-p 4566:4566   -p 4510-4559:4510-4559   -v ~/localstack-data:/var/lib/localstack   -v /var/run/docker.sock:/var/run/docker.sock   localstack/localstack
```

dengan ada nya variable 

```
LAMBDA_DOCKER_NETWORK=bridge
```

maka nantinya lambda dan container localtsack akan tau bagaimana bisa saling terhubung
