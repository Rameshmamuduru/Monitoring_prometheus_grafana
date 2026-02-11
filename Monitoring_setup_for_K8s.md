# Setup

## Phase-1

1️⃣ Helm + Prometheus + Grafana (basic)
2️⃣ Expose Grafana (Ingress)
3️⃣ Add Auth + TLS

## STEP 1 — Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

You MUST see helm version.

---

## STEP 2 — Create namespace

```bash
kubectl create namespace monitoring
```

---

## STEP 3 — Install kube-prometheus-stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring
```

WAIT 2 minutes.

---

## STEP 4 — Verify pods

```bash
kubectl get pods -n monitoring
```

You should see:

```
monitoring-grafana
monitoring-prometheus
alertmanager
node-exporter
kube-state-metrics
```

If NOT → copy output and send to me.

---

## STEP 5 — Access Grafana TEMPORARILY

Just to confirm it works:

```bash
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
```

Browser:

```
http://localhost:3000
```

Login:

```
admin
prom-operator
```

Perfect 👍 — that’s great.

So **Phase-1 is DONE**:

✅ Helm installed
✅ kube-prometheus-stack running
✅ Grafana opens via port-forward

Now we move to **Phase-2 (Expose Grafana using NGINX Ingress)** — slowly and cleanly.

---

# PHASE 2 — Install NGINX Ingress

### STEP 1 — Install ingress controller

Run:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.0/deploy/static/provider/cloud/deploy.yaml
```

Wait:

```bash
kubectl get pods -n ingress-nginx
```

All must be **Running**.

Tell me when done.

---

# STEP 2 — Get ingress IP

```bash
kubectl get svc -n ingress-nginx
```

You’ll see something like:

```
ingress-nginx-controller   LoadBalancer   <PRIVATE-IP>
```

Copy that IP.

---

# STEP 3 — Create Grafana Ingress (NO AUTH / NO TLS YET)

Create file:

```bash
nano grafana-ingress.yaml
```

Paste:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
spec:
  ingressClassName: nginx
  rules:
  - host: grafana.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: monitoring-grafana
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f grafana-ingress.yaml
```

---

# STEP 4 — Add hosts entry (LOCAL MACHINE)

Edit:

### Linux / Mac:

```bash
sudo nano /etc/hosts
```

Add:

```
INGRESS_IP grafana.local
```

Example:

```
10.0.2.15 grafana.local
```

Save.

---

# STEP 5 — Test Grafana via Ingress

Browser:

```
http://grafana.local
```

Grafana login:

```
admin
prom-operator
```

---
```

| Method            | Used in prod |
| ----------------- | ------------ |
| Internal ALB      | ✅            |
| Security Groups   | ✅            |
| Ingress whitelist | ✅            |
| VPN               | ✅            |
| Public access     | ❌            |
```


