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
```

| Method            | Used in prod |
| ----------------- | ------------ |
| Internal ALB      | ✅            |
| Security Groups   | ✅            |
| Ingress whitelist | ✅            |
| VPN               | ✅            |
| Public access     | ❌            |
```



# STEP 3 — Create DNS Record

# ===============================

Create A record:

```
grafana.yourdomain.com → ingress LB IP/DNS
```

( Route53 / Cloudflare / GoDaddy )

Wait 1–2 mins.

---

# ===============================

# STEP 4 — Install cert-manager

# ===============================

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.3/cert-manager.yaml
```

Verify:

```bash
kubectl get pods -n cert-manager
```

---

# ===============================

# STEP 5 — Create Let's Encrypt Issuer

# ===============================

```bash
nano issuer.yaml
```

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: you@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

Apply:

```bash
kubectl apply -f issuer.yaml
```

---

# STEP 7 — Create Basic Auth

```bash
sudo apt install apache2-utils -y
```

```bash
htpasswd -c auth admin
```

```bash
kubectl create secret generic grafana-auth \
--from-file=auth -n monitoring
```

---

# ===============================

# STEP 8 — Create Grafana Ingress (TLS + Auth)

# ===============================

```bash
nano grafana-ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: grafana-auth
    nginx.ingress.kubernetes.io/auth-realm: "Restricted"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - grafana.yourdomain.com
    secretName: grafana-tls
  rules:
  - host: grafana.yourdomain.com
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

# ===============================

# STEP 9 — Wait for TLS

# ===============================

```bash
kubectl get certificate -n monitoring
```

Wait until:

```
READY True
```

---

# ===============================

# STEP 10 — Access Grafana

# ===============================

Browser:

```
https://grafana.yourdomain.com
```

Login:

### NGINX:

admin / yourpassword

### Grafana:

admin / prom-operator

---

# 🎉 DONE — YOU NOW HAVE

✅ Prometheus real-time metrics
✅ Grafana dashboards
✅ NGINX ingress
✅ HTTPS
✅ Authentication
✅ DNS
✅ Production architecture

---

#### Note

```
Nodes → node-exporter → Prometheus
Pods → kubelet/cAdvisor → Prometheus
Objects → kube-state-metrics → Prometheus
Apps → ServiceMonitor → Prometheus

Prometheus → Grafana
```



