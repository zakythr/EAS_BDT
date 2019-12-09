# EAS Implementasi TiDb Cluster

#### Nama : Zaky Thariq H
#### NRP : 05111740000140

## Gambar Arsitektur

## Detail Aksitektur

1. Node 1
- IP : 192.168.17.140, OS : geerlingguy/CentOS7, Memory : 512 mb
- Service yang tersedia : Node Exporter, TiDB, PD, Grafana dan Prometheus

2. Node 2
- IP : 192.168.17.141, OS : geerlingguy/CentOS7, Memory : 512 mb
- Service yang tersedia : PD, Node Exporter

3. Node 3
- IP : 192.168.17.142, OS : geerlingguy/CentOS7, Memory : 512 mb
- Service yang tersedia : PD, Node Exporter

4. Node 4
- IP : 192.168.17.143, OS : geerlingguy/CentOS7, Memory : 512 mb
- Service yang tersedia : PD, Node Exporter

5. Node 5
- IP : 192.168.17.144, OS : geerlingguy/CentOS7, Memory : 512 mb
- Service yang tersedia : PD, Node Exporter

6. Node 6
- IP : 192.168.17.145, OS : geerlingguy/CentOS7, Memory : 512 mb
- Service yang tersedia : PD, Node Exporter

## Konfigurasi Vagrantfile

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    (1..6).each do |i|
      config.vm.define "node#{i}" do |node|
        node.vm.hostname = "node#{i}"

        # Gunakan CentOS 7 dari geerlingguy yang sudah dilengkapi VirtualBox Guest Addition
        node.vm.box = "geerlingguy/centos7"
        node.vm.box_version = "1.2.19"
        
        # Disable checking VirtualBox Guest Addition agar tidak compile ulang setiap restart
        node.vbguest.auto_update = false
        
        node.vm.network "private_network", ip: "192.168.17.#{139+i}"
        
        node.vm.provider "virtualbox" do |vb|
          vb.name = "node#{i}"
          vb.gui = false
          vb.memory = "512"
        end
  
        node.vm.provision "shell", path: "provision/bootstrap.sh", privileged: false
      end
    end
  end
```

## Provision

```
# Copy open files limit configuration
sudo cp /vagrant/config/tidb.conf /etc/security/limits.d/

# Enable max open file
sudo sysctl -w fs.file-max=1000000

# Copy atau download TiDB binary dari http://download.pingcap.org/tidb-v3.0-linux-amd64.tar.gz
cp /vagrant/installer/tidb-v3.0-linux-amd64.tar.gz .

# Extract TiDB binary
tar -xzf tidb-v3.0-linux-amd64.tar.gz

# Install MariaDB to get MySQL client
sudo yum -y install mariadb

# Install Git
sudo yum -y install git

# Install nano text editor
sudo yum -y install nano
```

## Konfigurasi Setiap Node
Lakukan di setiap node secara urut

```
Run on PD Server
//node 1
cd tidb-v3.0-linux-amd64
./bin/pd-server --name=pd1 --data-dir=pd --client-urls="http://192.168.17.140:21449" --peer-urls="http://192.168.17.140:21450" --initial-cluster="pd1=http://192.168.17.140:21450,pd2=http://192.168.17.141:21450,pd3=http://192.168.17.142:21450" --log-file=pd.log &
// node 2
cd tidb-v3.0-linux-amd64
./bin/pd-server --name=pd2 --data-dir=pd --client-urls="http://192.168.17.141:21449" --peer-urls="http://192.168.17.141:21450" --initial-cluster="pd1=http://192.168.17.140:21450,pd2=http://192.168.17.141:21450,pd3=http://192.168.17.142:21450" --log-file=pd.log &
// node 3
cd tidb-v3.0-linux-amd64
./bin/pd-server --name=pd3 --data-dir=pd --client-urls="http://192.168.17.142:21449" --peer-urls="http://192.168.17.142:21450" --initial-cluster="pd1=http://192.168.17.140:21450,pd2=http://192.168.17.141:21450,pd3=http://192.168.17.142:21450" --log-file=pd.log &
======================================================

Run on Tikv Server

//node 4
cd tidb-v3.0-linux-amd64
./bin/tikv-server --pd="192.168.17.140:21449,192.168.17.141:21449,192.168.17.142:21449" --addr="192.168.17.143:20170" --data-dir=tikv --log-file=tikv.log &

//node 5
cd tidb-v3.0-linux-amd64
./bin/tikv-server --pd="192.168.17.140:21449,192.168.17.141:21449,192.168.17.142:21449" --addr="192.168.17.144:20170" --data-dir=tikv --log-file=tikv.log &

//node 6
cd tidb-v3.0-linux-amd64
./bin/tikv-server --pd="192.168.17.140:21449,192.168.17.141:21449,192.168.17.142:21449" --addr="192.168.17.145:20170" --data-dir=tikv --log-file=tikv.log &

======================================================

Run on TiDB Server
//node 1 lagi
cd tidb-v3.0-linux-amd64
./bin/tidb-server --store=tikv --path="192.168.17.140:21449" --log-file=tidb.log &


====================================================================================================================================================================================================================================================================================================================================

untuk tidb single (untuk testing sysbench dengan 1 PD)

halt node2 dan node3

Run on PD Server

//node 1
cd tidb-v3.0-linux-amd64
./bin/pd-server --name=pd1 --data-dir=pd --client-urls="http://192.168.17.140:21449" --peer-urls="http://192.168.17.140:21450" --initial-cluster="pd1=http://192.168.17.140:21450" --log-file=pd.log &

======================================================

Run on Tikv Server

//node 4
cd tidb-v3.0-linux-amd64
./bin/tikv-server --pd="192.168.17.140:21449" --addr="192.168.17.143:20170" --data-dir=tikv --log-file=tikv.log &

//node 5
cd tidb-v3.0-linux-amd64
./bin/tikv-server --pd="192.168.17.140:21449" --addr="192.168.17.144:20170" --data-dir=tikv --log-file=tikv.log &

//node 6
cd tidb-v3.0-linux-amd64
./bin/tikv-server --pd="192.168.17.140:21449" --addr="192.168.17.145:20170" --data-dir=tikv --log-file=tikv.log &

======================================================

Run on TiDB Server

//node 1 lagi
cd tidb-v3.0-linux-amd64
./bin/tidb-server --store=tikv --path="192.168.17.140:21449" --log-file=tidb.log &
```

Selanjutnya

- Mengunjungi Node 1

```mysql -u root -h 192.168.17.140 -P 4000```
```create database (nama database)```

- Mengunjungi folder laravel dan melakukan migrate

```php artisan migrate```

## Pengoperasian Aplikasi

QurbanBerkah merupakan aplikasi penjualan hewan qurban sederhana yang menerapkan CRUD pada penambahan, pengeditan, dan penghapusan hewan qurban dari list penjualan.

- <h4>Konfigurasi Laravel</h4>
Pada .env kita ubah

![](/jpg_tidb/env.PNG)

- <h4>Login</h4>

![](/jpg_tidb/login.PNG)

- <h4>Menambah Hewan Qurban</h4>

![](/jpg_tidb/CREATE.PNG)

![](/jpg_tidb/afterCREATE.PNG)

- <h4>Mengedit Hewan Qurban</h4>

![](/jpg_tidb/EDIT.PNG)

![](/jpg_tidb/berhasilEDIT.PNG)

- <h4>Menghapus Hewan Qurban</h4>

![](/jpg_tidb/berhasilDELETE.PNG)

- <h4>Operasi Read Hewan Qurban</h4>

![](/jpg_tidb/READ.PNG)

## Pengujian dengan JMeter

- <h4>Dengan 100 koneksi</h4>

![](/jpg_tidb/jmeter100.PNG)

- <h4>Dengan 500 koneksi</h4>

![](/jpg_tidb/jmeter500.PNG)

- <h4>Dengan 1000 koneksi</h4>

![](/jpg_tidb/jmeter1000.PNG)

## Pengujian dengan Sysbench

Pengujian performa dari database dilakukan dengan menggunakan sysbench. 
Variasi pengujian diterapkan dengan jumlah PD cluster (1 PD cluster, 2 PD cluster, 3 PD cluster).

- Instal sysbench pada **Node 1**

```
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash
sudo yum -y install sysbench
```

- Selanjutnya mendownload file konfigurasi beserta aplikasi untuk testing tidb. Clone github :

```
git clone https://github.com/pingcap/tidb-bench.git
```

- Mulai mengkonfigurasi dengan mengedit file konfigurasi agar sesuai dengan koneksi tidb **Node1** pada file config

```
cd tidb-bench/sysbench
nano config
```

- Kemudian membuat database sbtest pada tidb (Node1). Masuk pada service mysql untuk membuat database sbtest.

```
mysql -h 172.5.17.20 -P 4000 -u root

CREATE DATABASE sbtest;
```

- Melakukan persiapan pengujian perintah

```./run.sh point_select prepare 100```

- Menjalankan pengujian

```./run.sh point_select run 100```

Hasil pengujian bisa dilihat pada file ```point_select_run_100.log```

Didapatkan hasil seperti dibawah ini

- Dengan 3 PD

![](/jpg_tidb/sysbench3pd.PNG)

- Dengan 2 PD

![](/jpg_tidb/sysbench2pd.PNG)

- Dengan 1 PD

![](/jpg_tidb/sysbench1pd.PNG)

**Kesimpulan** : Dengan melihat parameter rata-rata latency dan total query per detik, menunjukkan semakin banyak PD cluster yang digunakan, 
maka performa database akan semakin lebih cepat. Walaupun perubahan tersebut tidak terlalu besar.

## Monitoring Dashboard

### Instalasi Node Exporter

- Melakukan install Node Exporter pada semua Node (Node 1 - Node 6)

```
wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
tar -xzf node_exporter-0.18.1.linux-amd64.tar.gz
```

- Selanjutnya Node 1 sampai Node 6, jalankan perintah berikut

```
cd node_exporter-0.18.1.linux-amd64
./node_exporter --web.listen-address=":9100" \
    --log.level="info" &
```

### Instalasi Prometheus
- Pada Node 1 kita mendownload grafana dan promentheus, jalankan perintah di bawah ini

```
wget https://github.com/prometheus/prometheus/releases/download/v2.2.1/prometheus-2.2.1.linux-amd64.tar.gz
tar -xzf prometheus-2.2.1.linux-amd64.tar.gz
```

- Mengubah isi file **prometheus.yml** menjadi

```
global:
    scrape_interval:     15s  # By default, scrape targets every 15 seconds.
    evaluation_interval: 15s  # By default, scrape targets every 15 seconds.
    # scrape_timeout is set to the global default value (10s).
    external_labels:
        cluster: 'test-cluster'
        monitor: "prometheus"

scrape_configs:
- job_name: 'overwritten-nodes'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
    - '192.168.17.140:9100'
    - '192.168.16.34:9100'
    - '192.168.16.35:9100'
    - '192.168.16.36:9100'
    - '192.168.16.37:9100'
    - '192.168.16.38:9100'

- job_name: 'tidb'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
    - '192.168.16.33:10080'

- job_name: 'pd'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
    - '192.168.16.33:2379'
    - '192.168.16.34:2379'
    - '192.168.16.35:2379'

- job_name: 'tikv'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
    - '192.168.16.36:20180'
    - '192.168.16.37:20180'
    - '192.168.16.38:20180'
```

Menjalankan prometheus

```
./prometheus \
    --config.file="./prometheus.yml" \
    --web.listen-address=":9090" \
    --web.external-url="http://192.168.17.140:9090/" \
    --web.enable-admin-api \
    --log.level="info" \
    --storage.tsdb.path="./data.metrics" \
    --storage.tsdb.retention="15d" &
```

### Instalasi Grafana

```
wget https://dl.grafana.com/oss/release/grafana-6.5.1.linux-amd64.tar.gz
tar -zxf grafana-6.5.1.linux-amd64.tar.gz
```

- Ubah **grafana.ini**

```
[paths]
data = ./data
logs = ./data/log
plugins = ./data/plugins
[server]
http_port = 3000
domain = 192.168.17.140
[database]
[session]
[analytics]
check_for_updates = true
[security]
admin_user = admin
admin_password = admin
[snapshots]
[users]
[auth.anonymous]
[auth.basic]
[auth.ldap]
[smtp]
[emails]
[log]
mode = file
[log.console]
[log.file]
level = info
format = text
[log.syslog]
[event_publisher]
[dashboards.json]
enabled = false
path = ./data/dashboards
[metrics]
[grafana_net]
url = https://grafana.net
```

- Dan jalankan

```
./bin/grafana-server \
    --config="./conf/grafana.ini" &
```

- Masuk ke Dashboard Grafana dengan cara buka alamat IP node1 di browser pada port 3000 (192.168.17.16:3000)

![](/jpg_tidb/homeGRAFANA.PNG)

- Import Dashboard Grafana

![](/jpg_tidb/importGRAFANA.PNG)

- Hasil dari Grafana

Test-Cluster-PD

![](/jpg_tidb/grafana-clusterPD.PNG)

Test-Cluster-TiDb-Summary

![](/jpg_tidb/grafana-cluster-tidb-summary.PNG)

Test-Cluster-TiKv-Details

![](/jpg_tidb/cluster-tikv-details.PNG)

Test-Cluster-TiKv-Summary

![](/jpg_tidb/cluster-tikv-summary.PNG)


## Uji Coba Fail Over

- Langkah pertama buka url dibawah ini pada browser

```http://192.168.17.140:2379/pd/api/v1/members```

- Lalu muncul tampilan seperti di bawah ini

```
{
  "header": {
    "cluster_id": 6768374397138195061
  },
  "members": [
    {
      "name": "pd3",
      "member_id": 61451790943646900,
      "peer_urls": [
        "http://192.168.17.142:2380"
      ],
      "client_urls": [
        "http://192.168.17.142:2379"
      ]
    },
    {
      "name": "pd2",
      "member_id": 2200579136718574273,
      "peer_urls": [
        "http://192.168.17.141:2380"
      ],
      "client_urls": [
        "http://192.168.17.141:2379"
      ]
    },
    {
      "name": "pd1",
      "member_id": 14749030065058967029,
      "peer_urls": [
        "http://192.168.17.140:2380"
      ],
      "client_urls": [
        "http://192.168.17.140:2379"
      ]
    }
  ],
  "leader": {
    "name": "pd1",
    "member_id": 14749030065058967029,
    "peer_urls": [
      "http://192.168.17.140:2380"
    ],
    "client_urls": [
      "http://192.168.17.140:2379"
    ]
  },
  "etcd_leader": {
    "name": "pd1",
    "member_id": 14749030065058967029,
    "peer_urls": [
      "http://192.168.17.140:2380"
    ],
    "client_urls": [
      "http://192.168.17.140:2379"
    ]
  }
}
```

- Periksa leadernya berada di PD berapa, karena leader saat ini adalah node 1, maka service PD dimatikan pada node 1. Dengan cara di bawah ini


![](/jpg_tidb/failover_kill.PNG)


- Lalu buka browser lagi untuk memeriksa leader terbaru

Alamat IPnya dirubah, karena alamt IP yang sebelumya sudah dimatikan ```http://192.168.17.141:2379/pd/api/v1/members```

```
{
  "header": {
    "cluster_id": 6768374397138195061
  },
  "members": [
    {
      "name": "pd3",
      "member_id": 61451790943646900,
      "peer_urls": [
        "http://192.168.17.142:2380"
      ],
      "client_urls": [
        "http://192.168.17.142:2379"
      ]
    },
    {
      "name": "pd2",
      "member_id": 2200579136718574273,
      "peer_urls": [
        "http://192.168.17.141:2380"
      ],
      "client_urls": [
        "http://192.168.17.141:2379"
      ]
    },
    {
      "name": "pd1",
      "member_id": 14749030065058967029,
      "peer_urls": [
        "http://192.168.17.140:2380"
      ],
      "client_urls": [
        "http://192.168.17.140:2379"
      ]
    }
  ],
  "leader": {
    "name": "pd3",
    "member_id": 61451790943646900,
    "peer_urls": [
      "http://192.168.17.142:2380"
    ],
    "client_urls": [
      "http://192.168.17.142:2379"
    ]
  },
  "etcd_leader": {
    "name": "pd3",
    "member_id": 61451790943646900,
    "peer_urls": [
      "http://192.168.17.142:2380"
    ],
    "client_urls": [
      "http://192.168.17.142:2379"
    ]
  }
}
```

Hasilnya terlihat bahwa leader sekarang berad di PD3
