Yes — exactly 👍
Now that **hosts are monitored**, next logical step is:

👉 **Monitor Docker containers running on those same hosts.**

This is how real production setups do it.

Let’s keep it simple and practical.

---

# ✅ Final Monitoring Stack (Non-K8s Docker Hosts)

On **each server**:

| Component             | Port | Purpose           |
| --------------------- | ---- | ----------------- |
| Node Exporter         | 9100 | Host metrics      |
| cAdvisor              | 8080 | Container metrics |
| Docker Engine metrics | 9323 | Docker daemon     |

On **monitoring server**:

| Component    | Port |
| ------------ | ---- |
| Prometheus   | 9090 |
| Grafana      | 3000 |
| Alertmanager | 9093 |

---

# 🧱 Architecture

```
EC2 / VM
 ├── Node Exporter (9100) → CPU / RAM / Disk
 ├── cAdvisor (8080)     → Containers
 └── Docker metrics(9323)

              ↓
        Prometheus
              ↓
           Grafana
```

---

# 🟢 STEP 1 — Install cAdvisor (for containers)

On EVERY Docker host:

```bash
docker run -d \
--name=cadvisor \
--restart=always \
-p 8080:8080 \
-v /:/rootfs:ro \
-v /var/run:/var/run:ro \
-v /sys:/sys:ro \
-v /var/lib/docker/:/var/lib/docker:ro \
gcr.io/cadvisor/cadvisor:latest
```

Verify:

```
http://HOST_IP:8080
```

or

```bash
curl localhost:8080/metrics
```

✅ You’ll see container metrics.

---

# 🟢 STEP 2 — Enable Docker daemon metrics (9323)

Edit Docker config:

```bash
sudo nano /etc/docker/daemon.json
```

Add:

```json
{
  "metrics-addr": "0.0.0.0:9323",
  "experimental": true
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

Verify:

```bash
curl localhost:9323/metrics
```

---

# 🟢 STEP 3 — Open ports (AWS SG)

Allow from Prometheus IP ONLY:

```
9100
8080
9323
```

---

# 🟢 STEP 4 — Add targets in Prometheus

Edit `prometheus.yml`:

```yaml
scrape_configs:

- job_name: hosts
  static_configs:
  - targets:
    - "HOST_IP:9100"

- job_name: containers
  static_configs:
  - targets:
    - "HOST_IP:8080"

- job_name: docker
  static_configs:
  - targets:
    - "HOST_IP:9323"
```

Restart Prometheus.

Check:

```
http://PROMETHEUS:9090/targets
```

Everything must be **UP**.

---

# 🟢 STEP 5 — Grafana dashboards

Import:

### Containers:

```
193
```

### Docker:

```
1229
```

### Host:

```
1860
```

---

# 🎯 Now you can see:

### HOST:

CPU
RAM
Disk
Network

### CONTAINERS:

Per container CPU
Memory
Restarts
Network
Filesystem

### DOCKER:

Running containers
Images
Daemon health

---

# 🧠 Interview one-liner:

> We monitor VM hosts using Node Exporter, Docker containers using cAdvisor, Docker daemon via port 9323. Prometheus scrapes metrics and Grafana visualizes dashboards.

---

# 🔥 Production truth

For non-k8s:

✅ Node Exporter
✅ cAdvisor
✅ Docker metrics
✅ Prometheus
✅ Grafana
✅ Alertmanager

That’s all.

---

