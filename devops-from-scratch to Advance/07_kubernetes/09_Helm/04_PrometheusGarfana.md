Got it 👍 — you want **Prometheus + Grafana deployed in a custom namespace (`monitoring`)**. Here’s the correct and clean way to do it.

---

# 🚀 1. Create Namespace

```bash
kubectl create namespace monitoring
```

---

# 🔹 2. Add Helm Repo

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

# 📦 3. Install in `monitoring` Namespace

```bash
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```

👉 This installs everything inside the **monitoring namespace**

---

# 🔍 4. Verify Installation

```bash
kubectl get pods -n monitoring
```

```bash
kubectl get svc -n monitoring
```

---

# 🔐 5. Get Grafana Password

```bash
kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

---

# 🌐 6. Access Grafana

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Open:

```
http://localhost:3000
```

Login:

* Username: **admin**
* Password: (from above command)

---

# 📊 7. Access Prometheus

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090
```

Open:

```
http://localhost:9090
```

---

# ⚙️ 8. Optional: Use Custom values.yaml

```bash
cat <<EOF > monitor-values.yaml
grafana:
  service:
    type: NodePort

prometheus:
  service:
    type: NodePort
EOF
```

Install/Upgrade:

```bash
helm upgrade --install monitoring prometheus-community/kube-prometheus-stack -n monitoring -f monitor-values.yaml
```

---

# 🧹 9. Uninstall

```bash
helm uninstall monitoring -n monitoring
kubectl delete namespace monitoring
```

---

# ✅ Summary

* Namespace: **monitoring** ✔
* Prometheus + Grafana installed ✔
* Access via port-forward ✔
* Custom values supported ✔

---

