---
title: "Deploying PHP Guestbook application with Redis"
date: 2026-03-09
tags: [kubernetes, linux, Ci/Cd, devops, containerd,bare-metal,hybrid,ci,cd,ci/cd,system,infrastructure,php,database,]
header:
  teaser: /assets/images/php/peju.jpg
categories: [DevOps, Kubernetes,linux,Linux,Server,Deploy]
---
![asdasd](/assets/images/php/peju.jpg)

Jadi di post kali ini saya akan mengetes deployment dari sistem yang sudah lama saya buat yaitu sistem Ci/Cd, untuk tutorial step by stepnya 
[ada di sini](https://rizkiyusupk.github.io/devops/kubernetes/linux/server/hybird-ci-cd/) ,nah kurang lebih isi dari tutorial kali ini
itu memang cuman deploy sebuah php guest book dan hanya untuk test deployment semata dan satu hal lagi ini semua non production ready jadi ini semua itu cuman tes saja dan deployment ini itu multi tier deployment web jadi nanti isinya itu bukan cuman satu beberapa frontend web dengan satu instance redis ,tanpa berlama-lama lagi gas langsung ke tutorialnya

### **Software Versions:**
- **Kubernetes**: v1.28.15
- **Docker** : 28.2.2
- **Git Bash** : git version 2.51.1.windows.1
- **Jenkins**: 2.528.2
- **Container Runtime Interface**: containerd v2 2.1.3
- **Gitlab** : Web

### Deploy
pertama-tama kita harus memiliki repo gitlab terlebih dahulu agar nantinya bisa digunakan sebagai tempat untuk menyimpan semua configurasi dan bisa 
menggunakan webhook dari gitlab,buat terlebih dahulu sebuah repository di gitlab disini saya menggunakan repo yang sebelumnya sudah dibuat setelah itu clone repo ke local laptop pakai apapun bebas disini saya pakai git bash, nah jika sudah di clone 
saatnya membuat file configurasi,pertama-tama disini saya akan membuat file manifest untuk redis-leader-deployment dan servicenya terlebih dahulu

```
vim redis-leader-deployment.yaml
|
vim redis-leader-service.yaml
```

config

```
vim redis-leader-deployment.yaml
|
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-leader
  labels:
    app: redis
    role: leader
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        role: leader
        tier: backend
    spec:
      containers:
      - name: leader
        image: "registry.k8s.io/redis@sha256:cb111d1bd870a6a471385a4a69ad17469d326e9dd91e0e455350cacf36e1b3ee"
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
```

dan untuk servicenya

```
vim redis-leader-service.yaml
|
apiVersion: v1
kind: Service
metadata:
  name: redis-leader
  labels:
    app: redis
    role: leader
    tier: backend
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
    role: leader
    tier: backend
```

nah diatas merupakan config dari redis leader deployment dan servicenya,saya menggunakan image resminya,
untuk tahap selanjutnya itu bagian redis followernya sama seperti sebelumnya yaitu deployment dan service bersamaan

```
vim redis-follower-deployment.yaml
|
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-follower
  labels:
    app: redis
    role: follower
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        role: follower
        tier: backend
    spec:
      containers:
      - name: follower
        image: us-docker.pkg.dev/google-samples/containers/gke/gb-redis-follower:v2
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
```

selanjutnya

```
vim redis-follower-service.yaml
|
apiVersion: v1
kind: Service
metadata:
  name: redis-follower
  labels:
    app: redis
    role: follower
    tier: backend
spec:
  ports:
  - port: 6379
  selector:
    app: redis
    role: follower
    tier: backend
```

nah jika sudah kita masuk ke bagian selanjutnya yaitu frontend deployment dan servicenya juga


```
vim redis-frontend-deployment.yaml
|
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
        app: guestbook
        tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: us-docker.pkg.dev/google-samples/containers/gke/gb-frontend:v5
        env:
        - name: GET_HOSTS_FROM
          value: "dns"
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  ports:
  - port: 80
  selector:
    app: guestbook
    tier: frontend
```

untuk si frontend deployment dan servicenya disatukan engga dipisah, setelah itu masuk ke pembuatan jenkins file,

```
Jenkinsfile
|
pipeline {
    agent any
    stages {
        stage('Test Webhook') {
            steps {
                echo 'Webhook triggered successfully!'
                echo "Triger automatic deployment!!"
                echo "deploy"
                sh "kubectl apply -f redis-leader-deployment.yaml"
                sh "kubectl apply -f redis-leader-service.yaml"
                sh "kubectl apply -f redis-follower-deployment.yaml"
                sh "kubectl apply -f redis-follower-service.yaml"
                sh "kubectl apply -f redis-frontend-deployment.yaml"
            }
        }
    }
}
```

nah kurang lebih seperti diatas untuk Jenkinsfile, tahap selanjutnya tinggal deploy

```
git add .
|
git commit -m "add file"
|
git push origin main
```

tunggu sampai proses push selesai 

![asdhaud](/assets/images/php/Screenshot.png)

nah jika sudah bisa di cek di dashboard jenkins untuk melihat pipelinenya akses jangan lupa untuk mengaktifkan ngroknya

```
ip-node:portjenkins
|
192.168.100.7:9091
```
nah jika sudah maka akan ada tampilan seperti ini

![asdjaisf](/assets/images/errors/Screenshot 2026-03-02 195808.png)

masuk ke pipeline yang sudah dibuat sebelumnya di [tutorial ini](https://rizkiyusupk.github.io/devops/kubernetes/linux/server/hybird-ci-cd/)
masuk ke pipiline nya maka akan ada tampilan seperti ini 

![uadh](/assets/images/php/Screenshot 2026-03-11 001907.png)

jangan haraukan pipeline diatas karena itu pipeline yang lama sudah di buat dan sudah di pakai berkali-kali, lihat ke 
salah satu build yang baru saja muncul dan lihat pada console output pada build test nya jika berhasil maka akan ada output seperti ini

![asidhaw](/assets/images/php/Screenshot 2026-03-11 002206.png)

nah jika sudah coba akses dari browser dengan mencari di node mana web nya terdeploy dengan cara ketik command ini di node master

```
kubectl get pods -o wide
|
kubectl describe pod name-pod | grep Node
```

jika sudah maka akan ada output semacam ini

![asdka](/assets/images/php/Screenshot 2026-03-11 002928.png)

dan akses dari ip node nya 

```
ip-node:port-service
|
192.168.100.56:30001
```

dan ini outputnya

![aisdiad](/assets/images/php/Screenshot 2026-03-11 001228.png)

selesaiii sudah!!!
