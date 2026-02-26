## Simple Demo Docker Swarm

Project ini adalah demo Docker Swarm dengan 1 manager dan 2 worker.

Contoh implementasi Docker Swarm menggunakan Vagrant (Bisa diganti dengan Virtual Manchine seperi VMWare, Virtualbox dll) dengan membuat cluster lab lokal (1 manager + 2 worker).

Contoh ini cocok untuk:

* Lab DevOps
* Simulasi distributed system

### 🎯 Arsitektur Lab

Kita akan membuat 3 VM:

|VM|Role|IP|
|-|-|-|
|node1|Manager|192.168.33.10|
|node2|Worker|192.168.33.11|
|node3|Worker|192.168.33.12|

Semua VM berbasis:

* Alpine Linux

Virtualisasi menggunakan:

* Vagrant
* Provider: VirtualBox

Cluster orchestration:

* Docker Swarm

### 🏗 Struktur Folder

```
swarm-cluster/
│
├── manager/
│   └── Vagrantfile
│
├── worker1/
│   └── Vagrantfile
│
└── worker2/
    └── Vagrantfile
```

### 1️⃣ Vagrantfile – Manager

📁 `manager/Vagrantfile`

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "generic/alpine319"
  config.vm.hostname = "manager"
  config.vm.network "private\_network", ip: "192.168.33.10"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 1
  end

  config.vm.provision "shell", inline: <<-SHELL
    apk update
    apk add docker
    rc-update add docker boot
    service docker start
    adduser vagrant docker
  SHELL
end
```

### 2️⃣ Vagrantfile – Worker 1

📁 `worker1/Vagrantfile`

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "generic/alpine319"
  config.vm.hostname = "worker1"
  config.vm.network "private\_network", ip: "192.168.33.11"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 1
  end

  config.vm.provision "shell", inline: <<-SHELL
    apk update
    apk add docker
    rc-update add docker boot
    service docker start
    adduser vagrant docker
  SHELL
end
```

### 3️⃣ Vagrantfile – Worker 2

📁 `worker2/Vagrantfile`

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "generic/alpine319"
  config.vm.hostname = "worker2"
  config.vm.network "private\_network", ip: "192.168.33.12"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 1
  end

  config.vm.provision "shell", inline: <<-SHELL
    apk update
    apk add docker
    rc-update add docker boot
    service docker start
    adduser vagrant docker
  SHELL
end
```

### 🚀 Cara Menjalankan

Jalankan masing-masing secara terpisah:

```bash
cd manager
vagrant up

cd ../worker1
vagrant up

cd ../worker2
vagrant up
```

---

## 🧠 Inisialisasi Swarm

### 1️⃣ Masuk ke Manager

```bash
cd manager
vagrant ssh
```

Inisialisasi swarm:

```bash
docker swarm init --advertise-addr 192.168.33.10
```

Token akan keluar, copy token worker.

### 2️⃣ Join Worker

Masuk ke worker1:

```bash
cd worker1
vagrant ssh
```

Lalu:

```bash
docker swarm join --token <TOKEN> 192.168.33.10:2377
```

Ulangi di worker2.

### 🔍 Verifikasi

Di manager:

```bash
docker node ls
```

Harus muncul:

* manager → Leader
* worker1 → Ready
* worker2 → Ready

### 📦 Deploy Service

Di manager:

```bash
docker service create \\
  --name web \\
  --replicas 3 \\
  -p 8080:80 \\
  nginx:alpine
```

Cek distribusi:

```bash
docker service ps web
```

Cek node:

```bash
docker node ls
```

### 🔁 Uji Self-Healing

Matikan docker di worker:

```bash
service docker stop
```

Cek di manager:

```bash
docker service ps web
```

Swarm akan memindahkan container otomatis.

:::tip
Untuk melihat token yang ada manager jalankan perintah di bawah ini:

```bash
docker swarm join-token manager
```

:::

Jika service docker di jalankan lagi pada worker jangan lupa cek di manager statusnya:

```bash
docker node ls
```

