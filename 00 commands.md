# 🚀 Grafana & Prometheus – Commands & Scripts Guide

> Structured from **Basic → Advanced**, suitable for **DevOps labs, projects, and production reference**
> Covers: **Prometheus, Grafana, Node Exporter, Docker, Automation Scripts**

---

# 📌 Table of Contents

* [1. Prometheus Commands & Scripts](#1-prometheus-commands--scripts)
* [2. Grafana Commands & Scripts](#2-grafana-commands--scripts)
* [3. Docker Setup (Prometheus + Grafana)](#3-docker-setup-prometheus--grafana)

---

# 🎯 1. Prometheus Commands & Scripts

---

## ✅ 1.1 Basic Prometheus Commands

### Check Version

```bash
prometheus --version
```

### Start Prometheus (default)

```bash
./prometheus
```

### Start with Custom Config

```bash
./prometheus --config.file=prometheus.yml
```

### Validate Configuration

```bash
promtool check config prometheus.yml
```

### Validate Rules

```bash
promtool check rules rules.yml
```

### Validate Alert Rules

```bash
promtool check rules alert-rules.yml
```

---

## 🟦 1.2 Basic Setup Scripts

### Install Prometheus (Linux)

```bash
sudo useradd --no-create-home prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus

tar -xvf prometheus*.tar.gz
sudo cp prometheus promtool /usr/local/bin/
sudo cp -r consoles/ console_libraries/ prometheus.yml /etc/prometheus/
```

---

### Install Node Exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter.tar.gz
tar -xvf node_exporter.tar.gz
./node_exporter
```

---

## 🟧 1.3 Intermediate Commands

### Reload Config (No Restart)

```bash
curl -X POST http://localhost:9090/-/reload
```

### Check Targets

```bash
curl http://localhost:9090/api/v1/targets
```

### Service Status

```bash
systemctl status prometheus
```

### Manage Service

```bash
sudo systemctl start prometheus
sudo systemctl stop prometheus
sudo systemctl enable prometheus
```

---

## 🟥 1.4 Advanced Commands

### Create Systemd Service

```bash
sudo nano /etc/systemd/system/prometheus.service
```

```ini
[Unit]
Description=Prometheus Monitoring
After=network.target

[Service]
ExecStart=/usr/local/bin/prometheus \
--config.file=/etc/prometheus/prometheus.yml \
--storage.tsdb.path=/var/lib/prometheus

[Install]
WantedBy=multi-user.target
```

---

### View Rules & Alerts

```bash
curl http://localhost:9090/api/v1/rules
curl http://localhost:9093/api/v2/alerts
```

---

### PromQL Examples

#### CPU Usage %

```promql
100 - avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100
```

#### Memory Usage %

```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) 
/ node_memory_MemTotal_bytes * 100
```

#### Disk Free %

```promql
(node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100
```

---

## 🟪 1.5 Automation Scripts

### Backup Script

```bash
#!/bin/bash
DATA_DIR="/var/lib/prometheus"
BACKUP_DIR="/backup/prometheus-$(date +%F)"

mkdir -p $BACKUP_DIR
cp -r $DATA_DIR $BACKUP_DIR

echo "Backup Completed: $BACKUP_DIR"
```

---

### Safe Reload Script

```bash
#!/bin/bash

echo "Reloading Prometheus..."
curl -X POST http://localhost:9090/-/reload

if [ $? -ne 0 ]; then
    echo "Reload failed → Restarting service..."
    sudo systemctl restart prometheus
fi
```

---

# 🎯 2. Grafana Commands & Scripts

---

## 🟩 2.1 Basic Commands

### Check Version

```bash
grafana-server -v
```

### Start Service

```bash
sudo systemctl start grafana-server
```

### Enable on Boot

```bash
sudo systemctl enable grafana-server
```

### Check Status

```bash
sudo systemctl status grafana-server
```

---

## 🟦 2.2 Installation & Setup

### Install Grafana

```bash
sudo wget https://dl.grafana.com/oss/release/grafana_11.0.0_amd64.deb
sudo dpkg -i grafana_11.0.0_amd64.deb
sudo systemctl start grafana-server
```

---

### Reset Admin Password

```bash
grafana-cli admin reset-admin-password atul123
```

---

## 🟧 2.3 Intermediate Commands

### Restart Service

```bash
sudo systemctl restart grafana-server
```

### View Logs

```bash
sudo journalctl -u grafana-server -f
```

### Install Plugin

```bash
grafana-cli plugins install grafana-clock-panel
sudo systemctl restart grafana-server
```

---

## 🟥 2.4 Advanced API Usage

### Import Dashboard

```bash
curl -X POST -H "Content-Type: application/json" \
-d @dashboard.json \
http://admin:admin@localhost:3000/api/dashboards/db
```

---

### Add Data Source

```bash
curl -X POST http://admin:admin@localhost:3000/api/datasources \
-H "Content-Type: application/json" \
-d '{
  "name":"Prometheus",
  "type":"prometheus",
  "url":"http://localhost:9090",
  "access":"proxy",
  "basicAuth":false
}'
```

---

## 🟪 2.5 Automation Scripts

### Backup Dashboards

```bash
#!/bin/bash

DATE=$(date +%F)
BACKUP_DIR="/backup/grafana/$DATE"
mkdir -p $BACKUP_DIR

curl -s http://admin:admin@localhost:3000/api/search | jq -c '.[]' |
while read dashboard; do
    uid=$(echo $dashboard | jq -r '.uid')
    curl -s http://admin:admin@localhost:3000/api/dashboards/uid/$uid \
    > "$BACKUP_DIR/$uid.json"
done

echo "Grafana Backup Completed: $BACKUP_DIR"
```

---

### Auto-Provision Dashboards

```bash
cp -r dashboards/ /var/lib/grafana/dashboards/
systemctl restart grafana-server
```

---

# 🔥 3. Docker Setup (Prometheus + Grafana)

---

### Run Prometheus

```bash
docker run -d -p 9090:9090 \
-v $PWD/prometheus.yml:/etc/prometheus/prometheus.yml \
prom/prometheus
```

---

### Run Grafana

```bash
docker run -d -p 3000:3000 \
--name=grafana grafana/grafana
```

---

### Run Node Exporter

```bash
docker run -d -p 9100:9100 prom/node-exporter
```

---

# 🏁 Conclusion

This guide provides:

* ✅ **End-to-end commands (Basic → Advanced)**
* ✅ **Production-ready scripts**
* ✅ **Automation + Backup strategies**
* ✅ **Docker-based deployment**

Perfect for:

* DevOps Projects
* Monitoring Labs
* Production Setup Documentation
