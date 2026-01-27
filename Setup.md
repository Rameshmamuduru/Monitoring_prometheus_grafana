# End-to-End Server Monitoring Setup (From Scratch)

This document explains how to set up **Prometheus, Node Exporter, Grafana, and Alertmanager** for **server-level monitoring (non-Kubernetes)** in a **real-time, production-style** environment.

---

## 1. Architecture Overview

```
[ App / DB Servers ]
   |  (Node Exporter :9100)
   |
   v   (PULL model)
[ Prometheus Server :9090 ]
   |
   v
[ Alertmanager :9093 ]
   |
   v
[ Grafana UI :3000 ]
```

### Key Concepts

* **Node Exporter** exposes metrics
* **Prometheus pulls metrics** (never push)
* **Grafana visualizes metrics**
* **Alertmanager handles alerts**

---

## 2. Prerequisites

* Ubuntu servers
* Network connectivity between monitoring server and app servers
* Ports allowed:

  * 9100 (Node Exporter)
  * 9090 (Prometheus)
  * 9093 (Alertmanager)
  * 3000 (Grafana)

---

## 3. Install Node Exporter (On All Target Servers)

### 3.1 Create User

```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```

### 3.2 Download and Install

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node_exporter-1.7.0.linux-amd64.tar.gz
sudo cp node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
```

### 3.3 Create systemd Service

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

### 3.4 Verify

```
http://<SERVER-IP>:9100/metrics
```

---

## 4. Install Prometheus (Monitoring Server)

### 4.1 Create User

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

### 4.2 Download & Install

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.51.0/prometheus-2.51.0.linux-amd64.tar.gz
tar xvf prometheus-2.51.0.linux-amd64.tar.gz
sudo cp prometheus-2.51.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.51.0.linux-amd64/promtool /usr/local/bin/
```

### 4.3 Create Directories

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

### 4.4 Configuration (`prometheus.yml`)

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "localhost:9093"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"

  - job_name: "node_exporters"
    static_configs:
      - targets:
          - "10.0.0.21:9100"
          - "10.0.0.22:9100"
```

### 4.5 systemd Service

```ini
[Unit]
Description=Prometheus
After=network-online.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

### 4.6 Verify

```
http://<PROM-IP>:9090
Status → Targets
```

---

## 5. Install Alertmanager

### 5.1 Download & Install

```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
tar xvf alertmanager-0.27.0.linux-amd64.tar.gz
sudo cp alertmanager-0.27.0.linux-amd64/alertmanager /usr/local/bin/
sudo cp alertmanager-0.27.0.linux-amd64/amtool /usr/local/bin/
```

### 5.2 Configuration

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: "default"

receivers:
  - name: "default"
```

### 5.3 systemd Service

```ini
[Unit]
Description=Alertmanager
After=network.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```

### 5.4 Verify

```
http://<PROM-IP>:9093
```

---

## 6. Install Grafana

```bash
sudo apt update
sudo apt install -y apt-transport-https software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install -y grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### Access

```
http://<PROM-IP>:3000
admin / admin
```

---

## 7. Grafana Configuration

### 7.1 Add Prometheus Data Source

* URL: `http://localhost:9090`
* Save & Test

### 7.2 Import Dashboard

* Dashboard ID: **1860** (Node Exporter Full)

---

## 8. Validation Checklist

* Prometheus targets are UP
* Node Exporter reachable on :9100
* Grafana dashboards show data
* Alertmanager UI accessible

---

## 9. Real-Time Production Best Practices

* Dedicated monitoring server
* Least-privilege users
* Firewall restricts ports
* Alerts via Slack / Email
* Backup Prometheus data

---

## 10. Interview Summary

> "Prometheus pulls metrics from Node Exporter, Alertmanager handles alerts, and Grafana visualizes metrics. All components run as systemd services on a dedicated monitoring server."

---

## 11. Next Enhancements

* Alert rules (CPU, Disk)
* Slack / Email integration
* TLS & authentication
* High availability (Thanos)
