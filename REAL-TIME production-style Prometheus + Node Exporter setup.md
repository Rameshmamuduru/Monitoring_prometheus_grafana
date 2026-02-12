Got it — now you’re asking:

👉 **“Give me the complete setup from scratch”**

Let’s do a **REAL-TIME production-style Prometheus + Node Exporter setup** for your case:

You said earlier:

✅ App Server 1
✅ App Server 2
✅ Jenkins Server
✅ One Monitoring Server (Prometheus + Grafana)

Goal:

👉 Monitor ALL servers.

---

# 🏗 Architecture

```
Monitoring VM
 ├── Prometheus
 ├── Grafana
 └── Node Exporter

App Server 1
 └── Node Exporter

App Server 2
 └── Node Exporter

Jenkins Server
 ├── Jenkins
 └── Node Exporter
```

---

# 🔵 STEP-1 (ALL SERVERS)

Install Node Exporter on:

✔ App1
✔ App2
✔ Jenkins
✔ Monitoring VM

---

## On EACH server:

```bash
sudo useradd --no-create-home --shell /bin/false node_exporter

cd /opt
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz

tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz

sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
```

Create service:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Paste:

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

Verify:

```
http://SERVER_IP:9100/metrics
```

---

# 🔵 STEP-2 (Monitoring Server ONLY)

Install Prometheus.

---

```bash
sudo useradd --no-create-home --shell /bin/false prometheus

cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.50.0/prometheus-2.50.0.linux-amd64.tar.gz

tar -xvf prometheus-2.50.0.linux-amd64.tar.gz
```

---

### Create directories:

```bash
sudo mkdir /etc/prometheus
sudo mkdir /data
```

---

### Move files:

```bash
sudo mv prometheus-2.50.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.50.0.linux-amd64/promtool /usr/local/bin/

sudo mv prometheus-2.50.0.linux-amd64/consoles /etc/prometheus/
sudo mv prometheus-2.50.0.linux-amd64/console_libraries /etc/prometheus/
sudo mv prometheus-2.50.0.linux-amd64/prometheus.yml /etc/prometheus/
```

---

# 🔵 STEP-3 Configure Targets

Edit:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Replace with:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'nodes'
    static_configs:
      - targets:
          - 'APP1_IP:9100'
          - 'APP2_IP:9100'
          - 'JENKINS_IP:9100'
          - 'MONITORING_IP:9100'
```

---

# 🔵 STEP-4 Prometheus Service

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Paste:

```ini
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
--config.file=/etc/prometheus/prometheus.yml \
--storage.tsdb.path=/data

[Install]
WantedBy=multi-user.target
```

Start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

---

Verify:

```
http://MONITORING_IP:9090
```

---

# 🔵 STEP-5 Install Grafana (Monitoring VM)

```bash
sudo apt install -y grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Access:

```
http://MONITORING_IP:3000
```

---

Login:

```
admin / admin
```

Add Prometheus datasource:

```
http://localhost:9090
```

Import dashboard:

```
1860
```

(Node Exporter Full)

---

# ✅ DONE

You now have:

✔ Server metrics
✔ CPU
✔ RAM
✔ Disk
✔ Network
✔ Jenkins machine health

---

# 🎯 Real Production Checklist

Open security groups:

```
9100  Node exporter
9090  Prometheus
3000  Grafana
```

---

If you want next:

✅ Alertmanager setup
✅ Email alerts
✅ Disk full alerts
✅ Jenkins job alerts

Just tell 👍
