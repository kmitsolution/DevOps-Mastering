Here’s a **simple, working guide to deploy the Kubernetes Dashboard** on your cluster 👇

---

# 🚀 1. Deploy Kubernetes Dashboard

```bash
# Apply official dashboard manifest
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

---

# 🔍 2. Verify Deployment

```bash
# Check all resources in kubernetes-dashboard namespace
kubectl get all -n kubernetes-dashboard
```

You should see:

* pod: dashboard
* service: kubernetes-dashboard
* secret, configmap, etc.

---

# 🔐 3. Create Admin User (Service Account)

Create a file:

```bash
cat <<EOF > dashboard-admin.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

Apply it:

```bash
kubectl apply -f dashboard-admin.yaml
```

---

# 🔑 4. Get Login Token

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

👉 Copy this token — you’ll use it to log in.

---

# 🌐 5. Access Dashboard

### Option A: Port Forward (recommended)

```bash
kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8443:443
```

Now open:

```
https://localhost:8443
```

---

### Option B: NodePort (optional)

```bash
kubectl patch svc kubernetes-dashboard -n kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}'
kubectl get svc -n kubernetes-dashboard
```

Then access via:

```
https://<NODE-IP>:<NODE-PORT>
```

---

# 🔓 6. Login

* Select **Token**
* Paste token from Step 4
* Click **Sign in**

---

# ⚠️ Notes

* You may see a browser warning (self-signed cert) → proceed anyway
* Dashboard requires RBAC enabled (default in most clusters)
* Use **port-forward** for safer local access

---

# ✅ Quick Checklist

* Dashboard deployed ✔
* Admin user created ✔
* Token generated ✔
* Access via browser ✔

