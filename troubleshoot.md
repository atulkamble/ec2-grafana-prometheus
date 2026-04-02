# 📊 Prometheus + Node Exporter + Grafana Setup (Fix: No Metrics Issue)

## 🔍 Problem

Grafana dashboard shows **no metrics**

## 🎯 Root Cause

Prometheus was **not scraping Node Exporter**

---

# ✅ Step 1: Verify Node Exporter

```bash
ps -ef | grep node_exporter
```

Test endpoint:

```bash
curl http://localhost:9100/metrics
```

---

# ✅ Step 2: Update Prometheus Config

Edit config file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Replace with:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

---

# ✅ Step 3: Restart Prometheus

```bash
sudo systemctl restart prometheus
sudo systemctl status prometheus
```

✔ Ensure:

```
Active: active (running)
```

---

# ✅ Step 4: Verify Targets

Open:

```
http://<EC2-IP>:9090/targets
```

✔ Expected:

* prometheus → UP
* node_exporter → UP

---

# ✅ Step 5: Test Metrics

In Prometheus UI:

```
http://<EC2-IP>:9090
```

Run query:

```
up
```

---

# ✅ Step 6: Configure Grafana

* Data Source → Prometheus
* URL: `http://localhost:9090`

Import Dashboard:

```
ID: 1860 (Node Exporter Dashboard)
```

---

# ⚡ Troubleshooting

| Issue           | Fix                             |
| --------------- | ------------------------------- |
| No metrics      | Add node_exporter in Prometheus |
| Service failed  | Fix YAML syntax                 |
| Empty dashboard | Check time range                |
| Target DOWN     | Check port 9100                 |

---

# 🧠 Architecture

```
Node Exporter → Prometheus → Grafana
```

---

# 🚀 Result

✔ System metrics visible in Grafana
✔ CPU, Memory, Disk, Network monitoring working

---
