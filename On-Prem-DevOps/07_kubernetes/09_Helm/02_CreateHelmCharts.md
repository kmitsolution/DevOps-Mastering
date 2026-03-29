Here’s a clean, step-by-step version of everything you listed, organized so you can follow it like a mini Helm lab 🚀

---

# 🔹 1. Add & Manage Helm Repositories

```bash
# Add stable repo
helm repo add stable https://charts.helm.sh/stable

# Search stable repo
helm search repo stable

# Remove stable repo
helm repo remove stable

# Add Bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# Search Bitnami charts
helm search repo bitnami
```

---

# 🔹 2. Install a Chart (Apache)

```bash
# Install Apache chart
helm install apache bitnami/apache

# Verify resources
kubectl get all
```

---

# 🔹 3. Create Your Own Chart

```bash
# Create chart
helm create helloworld
```

---

# 🔹 4. Install Custom Chart

```bash
# Install chart
helm install helloworld-1 helloworld

# Check deployments
kubectl get deploy
```

---

# 🔹 5. Override Values via CLI

```bash
# Install with replicaCount = 2
helm install --set replicaCount=2 helloworld-2 helloworld

# Verify
kubectl get deploy
```

---

# 🔹 6. Upgrade Release

```bash
# Upgrade replica count to 4
helm upgrade --set replicaCount=4 helloworld-2 helloworld
```

---

# 🔹 7. Override Using values.yaml File

```bash
# Create custom values file
cat <<EOF > myvalues.yaml
replicaCount: 5
EOF

# Upgrade using file
helm upgrade -f myvalues.yaml helloworld-2 helloworld
```

---

# 🔹 8. Create Another Chart (mychart)

```bash
# Create chart
helm create mychart

# Render templates locally
helm template mychart

# Lint chart
helm lint mychart
```

---

# 🔹 9. Modify Templates

```bash
# Go to templates folder
cd mychart/templates

# Remove default templates
rm -rf *

# Create your own YAML file
touch pod.yaml
```

👉 Example `pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod20
spec:
  containers:
    - name: nginx
      image: nginx
```

---

# 🔹 10. Install Your Custom Chart

```bash
cd ..

helm install mychart ./mychart
```

---

# 🔹 11. Verify Deployment

```bash
# List Helm releases
helm list

# Check pods (pod20 should be created)
kubectl get pods
```

---

# 🔹 12. Uninstall Chart

```bash
helm uninstall mychart
```

---

# ✅ Summary

You just covered:

* Adding/removing Helm repos
* Installing charts (Bitnami Apache)
* Creating and deploying custom charts
* Overriding values (CLI + YAML)
* Upgrading releases
* Building a chart from scratch
* Cleaning up resources

---
