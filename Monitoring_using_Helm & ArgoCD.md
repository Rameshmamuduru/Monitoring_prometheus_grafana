Perfect 😄 — here’s a **full, ready-to-use GitOps setup** for **monitoring your existing applications** with **Prometheus + Grafana** using **Helm + Argo CD**, with clear folder structure and code you can commit to your Git repo.

---

## **1️⃣ Git Repo Structure**

```
k8s-gitops-repo/
├── apps/
│   └── my-app/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── ingress.yaml
├── monitoring/                # resources for monitoring stack
│   ├── values.yaml
│   ├── prometheus-rules/
│   │   └── cpu-alert.yaml
│   ├── service-monitors/
│   │   └── my-app-servicemonitor.yaml
│   └── dashboards/
│       └── my-dashboard.yaml
└── argocd/                    # Argo CD application manifests
    └── monitoring-app.yaml
```

---

## **2️⃣ `monitoring/values.yaml`** (Helm overrides)

```yaml
grafana:
  adminPassword: "StrongPassword123"

alertmanager:
  alertmanagerSpec:
    secrets:
      - alertmanager-slack
  config:
    global:
      resolve_timeout: 5m
    route:
      receiver: slack-notifications
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 4h
    receivers:
      - name: slack-notifications
        slack_configs:
          - api_url_file: /etc/alertmanager/secrets/alertmanager-slack/slack-webhook-url
            channel: "#alerts"
            send_resolved: true

prometheus:
  prometheusSpec:
    retention: 15d
```

---

## **3️⃣ PrometheusRules (monitoring/prometheus-rules/cpu-alert.yaml)**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: high-cpu
  namespace: monitoring
spec:
  groups:
  - name: cpu-alerts
    rules:
    - alert: HighCPUUsage
      expr: avg(rate(container_cpu_usage_seconds_total[2m])) > 0.8
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "High CPU usage"
        description: "CPU usage > 80% for 1 minute"
```

---

## **4️⃣ ServiceMonitor (monitoring/service-monitors/my-app-servicemonitor.yaml)**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: my-app       # matches your app Service labels
  endpoints:
    - port: http-metrics  # port your app exposes metrics
      interval: 30s
```

---

## **5️⃣ Dashboard Example (monitoring/dashboards/my-dashboard.yaml)**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "1"
data:
  my-dashboard.json: |
    {
      "dashboard": {
        "id": null,
        "title": "My App Dashboard",
        "panels": [],
        "schemaVersion": 16,
        "version": 0
      }
    }
```

---

## **6️⃣ Argo CD Application YAML (argocd/monitoring-app.yaml)**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: monitoring
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/yourcompany/k8s-gitops-repo.git
    targetRevision: main
    path: monitoring
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

> This will deploy **Prometheus stack + your custom ServiceMonitor, PrometheusRules, dashboards** automatically.

---

## **7️⃣ Secret for Slack Alerts (monitoring/secrets/alertmanager-slack.yaml)**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: alertmanager-slack
  namespace: monitoring
type: Opaque
stringData:
  slack-webhook-url: "https://hooks.slack.com/services/T000/B000/XXXX"
```

---

## **8️⃣ How it works**

1. Commit all folders (`apps/`, `monitoring/`, `argocd/`) to Git.
2. Apply the Argo CD application:

```bash
kubectl apply -f argocd/monitoring-app.yaml -n argocd
```

3. Argo CD will:

   * Render Helm chart using `values.yaml` → deploy Prometheus, Grafana, Alertmanager, node-exporter, kube-state-metrics
   * Apply custom PrometheusRules, ServiceMonitors, dashboards
   * Sync automatically on changes

4. Prometheus scrapes your **existing applications** metrics, Grafana shows dashboards, alerts go to Slack.

---

✅ **Production-ready benefits**

* Apps stay plain YAML → no Helm required for your app.
* Helm is only used for monitoring stack.
* Fully declarative GitOps → Argo CD manages everything.
* Scalable to hundreds of apps & nodes.

---

If you want, I can also **draw a diagram showing Git repo → Argo CD → Cluster → Prometheus / Grafana → Slack**, which is perfect for **documentation**.

Do you want me to do that next?
