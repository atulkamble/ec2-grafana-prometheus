# 📊 Grafana & Prometheus Installation on AWS EC2

## 🔹 Environment Details

| Component     | Value          |
| ------------- | -------------- |
| Cloud         | AWS            |
| Service       | EC2            |
| Instance Type | `t3.medium`    |
| OS            | Amazon Linux 2 |
| Key Pair      | `grafana.pem`  |
| User          | `ec2-user`     |

---

![Image](https://d2908q01vomqb2.cloudfront.net/ca3512f4dfa95a03169c5a670a4c91a19b3077b4/2021/04/19/koffir_prometheus_monitor_ec2_f1_new.png)

![Image](https://prometheus.io/assets/docs/grafana_qps_graph.png)

![Image](https://eadn-wc03-4064062.nxedge.io/cdn/wp-content/uploads/2021/06/Prometheus-Server_Chart.png)

![Image](https://media.licdn.com/dms/image/v2/D5612AQGmaf97dCRh-w/article-cover_image-shrink_720_1280/article-cover_image-shrink_720_1280/0/1681294143595?e=2147483647\&t=WQV1V_z5Te9L7B6JRAggbtQBZKO_Wdxjo80gTF7zusc\&v=beta)

---

## 🧩 Architecture Overview

* **Grafana** → Visualization layer (Port `3000`)
* **Prometheus** → Metrics collection & storage (Port `9090`)
* **EC2** → Hosts both services
* **Systemd** → Manages Prometheus & Grafana as services

---

## 🔹 Step 1: Update EC2 System

```bash
sudo yum update -y
```

---

## 🔹 Step 2: Install Grafana (Enterprise Edition)

### Install Grafana RPM

```bash
sudo yum install -y https://dl.grafana.com/grafana-enterprise/release/12.3.1/grafana-enterprise_12.3.1_20271043721_linux_amd64.rpm
```

### Verify Grafana Version

```bash
grafana-server --version
```

---

## 🔹 Step 3: Start & Enable Grafana Service

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

✅ **Grafana Web UI**

```
http://<EC2-PUBLIC-IP>:3000
```

**Default Credentials**

* Username: `admin`
* Password: `admin`

---

## 🔹 Step 4: Copy Prometheus Binary to EC2

### SCP from Local Machine (macOS/Linux)

```bash
scp -i /Users/atul/Downloads/grafana.pem \
/Users/atul/Downloads/prometheus-3.9.0-rc.0.linux-amd64.tar.gz \
ec2-user@ec2-98-89-6-93.compute-1.amazonaws.com:/home/ec2-user/
```

---

## 🔹 Step 5: Move & Extract Prometheus

```bash
sudo mv prometheus-3.9.0-rc.0.linux-amd64.tar.gz /opt
cd /opt
sudo tar -xvf prometheus-3.9.0-rc.0.linux-amd64.tar.gz
sudo mv prometheus-3.9.0-rc.0.linux-amd64 prometheus
sudo rm prometheus-3.9.0-rc.0.linux-amd64.tar.gz
```

---

## 🔹 Step 6: Create Prometheus System User

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

---

## 🔹 Step 7: Configure Prometheus Directories

```bash
cd /opt/prometheus

sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/

sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

sudo cp prometheus.yml /etc/prometheus/
```

### Set Permissions

```bash
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

---

## 🔹 Step 8: Create Prometheus systemd Service

```bash
sudo nano /etc/systemd/system/prometheus.service
```

### 📄 Paste the Following Configuration

```ini
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

---

## 🔹 Step 9: Start Prometheus Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

✅ **Prometheus Web UI**

```
http://<EC2-PUBLIC-IP>:9090
```

---

## 🔹 Step 10: AWS Security Group Configuration

| Service    | Port   | Protocol |
| ---------- | ------ | -------- |
| Grafana    | `3000` | TCP      |
| Prometheus | `9090` | TCP      |
| SSH        | `22`   | TCP      |

---

## 🎯 Final Validation Checklist

✔ Grafana running
✔ Prometheus running
✔ systemd services enabled
✔ Ports opened in Security Group
✔ Accessible via browser

---

## 🚀 Next Steps (Recommended)

* Add **Prometheus as Grafana Data Source**
* Install **Node Exporter**
* Import **Grafana Dashboards**
* Enable **Alertmanager**
* Add **TLS + Authentication**
* Convert setup into **Terraform / Ansible**

---

## 👨‍💻 Author

**Atul Kamble**

- 💼 [LinkedIn](https://www.linkedin.com/in/atuljkamble)
- 🐙 [GitHub](https://github.com/atulkamble)
- 🐦 [X](https://x.com/Atul_Kamble)
- 📷 [Instagram](https://www.instagram.com/atuljkamble)
- 🌐 [Website](https://www.atulkamble.in)


---

