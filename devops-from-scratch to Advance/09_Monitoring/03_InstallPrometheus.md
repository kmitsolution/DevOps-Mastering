 **Lesson 3: Install Prometheus + Grafana in Kubernetes using Helm**

Then we'll learn how to monitor a local server in a later step.

---

# Lesson 3 — Install Prometheus + Grafana Using Helm

For Kubernetes, we'll use the **kube-prometheus-stack** approach.

It gives us a complete monitoring stack:

```text
                    kube-prometheus-stack
                            │
            ┌───────────────┼────────────────┐
            │               │                │
            ▼               ▼                ▼
       Prometheus       Grafana         Alertmanager
            │
            │
     ┌──────┼───────────┐
     ▼      ▼           ▼
   Nodes   Pods    Kubernetes Objects
```

It also brings in components such as:

```text
Prometheus Operator
kube-state-metrics
Node Exporter
Prometheus
Grafana
Alertmanager
```

This is much closer to how you will encounter Prometheus in Kubernetes environments.

---

# 1. First Check Your Kubernetes Cluster

Run:

```bash
kubectl get nodes
```

You should see something like:

```text
NAME       STATUS   ROLES           AGE
kmaster    Ready    control-plane   20d
knode1     Ready    <none>          20d
knode2     Ready    <none>          20d
```

Then:

```bash
kubectl get pods -A
```

Make sure your cluster is healthy.

---

# 2. Check Helm

Run:

```bash
helm version
```

You should get something similar to:

```text
version.BuildInfo{
    Version:"v3.x.x"
}
```

If Helm isn't installed, we'll install it before proceeding.

---

# 3. Add the Prometheus Community Repository

We'll use the Prometheus Community Helm repository.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Then:

```bash
helm repo update
```

You can verify:

```bash
helm repo list
```

You should see:

```text
NAME                   URL
prometheus-community   https://prometheus-community.github.io/helm-charts
```

---

# 4. Create a Monitoring Namespace

I recommend keeping monitoring components in their own namespace.

```bash
kubectl create namespace monitoring
```

Verify:

```bash
kubectl get namespaces
```

You should see:

```text
monitoring
```

---

# 5. Install kube-prometheus-stack

Now the important command:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring
```

Here:

```text
monitoring
```

is our Helm release name.

And:

```text
prometheus-community/kube-prometheus-stack
```

is the chart.

---

# 6. Check the Installation

Run:

```bash
helm list -n monitoring
```

You should see something like:

```text
NAME        NAMESPACE    STATUS
monitoring  monitoring   deployed
```

Now:

```bash
kubectl get pods -n monitoring
```

You should see several pods.

For example:

```text
NAME                                                     READY
alertmanager-monitoring-kube-prometheus-alertmanager-0   2/2
monitoring-grafana                                      3/3
monitoring-kube-prometheus-operator                     1/1
monitoring-kube-state-metrics                            1/1
monitoring-prometheus-node-exporter-xxxxx                1/1
prometheus-monitoring-kube-prometheus-prometheus-0       2/2
```

**Don't worry if the exact names are different.**

The important thing is that the components are running.

---

# 7. Understand What We Just Installed

This is important.

We didn't just install Prometheus.

We installed an entire monitoring ecosystem:

```text
                 monitoring namespace
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   Prometheus         Grafana        Alertmanager
        │
        │
        ├──── kube-state-metrics
        │
        └──── Node Exporter
```

And the Prometheus Operator manages Prometheus through Kubernetes resources.

---

# 8. Check Services

Run:

```bash
kubectl get svc -n monitoring
```

You should see services similar to:

```text
NAME                                      TYPE
monitoring-grafana                        ClusterIP
monitoring-kube-prometheus-alertmanager   ClusterIP
monitoring-kube-prometheus-prometheus     ClusterIP
monitoring-kube-prometheus-operator      ClusterIP
```

Notice something important:

```text
TYPE = ClusterIP
```

That means these services aren't directly accessible from your machine.

We can temporarily use **port-forwarding**.

---

# 9. Access Grafana

Run:

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

You should see:

```text
Forwarding from 127.0.0.1:3000 -> 3000
```

Then open:

```text
http://localhost:3000
```

You should see Grafana.

---

# 10. Get the Grafana Password

The Helm chart creates a Kubernetes Secret.

Run:

```bash
kubectl get secret -n monitoring monitoring-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode
```

You'll get the password.

The default username is typically:

```text
admin
```

So:

```text
Username: admin
Password: <password from secret>
```

---

# 11. Access Prometheus

Open another terminal.

Run:

```bash
kubectl port-forward -n monitoring \
  svc/monitoring-kube-prometheus-prometheus \
  9090:9090
```

Then open:

```text
http://localhost:9090
```

Now you have the Prometheus UI.

---

# 12. Your Architecture Now

You have gone from:

```text
Kubernetes
```

to:

```text
Kubernetes
      │
      ▼
Helm
      │
      ▼
kube-prometheus-stack
      │
      ├──────────────┐
      │              │
      ▼              ▼
 Prometheus        Grafana
      │
      ├── Node Exporter
      │
      └── kube-state-metrics
```

That's a **much better starting point** for your goal.

---

# 13. Can This Monitor Your Local Server?

### YES.

But there is an important networking concept.

Suppose you have:

```text
Your Local Server
192.168.1.50
```

and your Kubernetes cluster is:

```text
Kubernetes Cluster
10.0.0.0/24
```

Install **Node Exporter** on the local server:

```text
Local Server
192.168.1.50
      │
      ▼
Node Exporter
      │
      │ :9100
      ▼
Prometheus
inside Kubernetes
```

Prometheus can scrape:

```text
192.168.1.50:9100
```

**if the Kubernetes nodes/pods have network connectivity to that server and TCP port 9100 is allowed.**

So Helm doesn't restrict Prometheus to Kubernetes.

Prometheus can monitor:

```text
Kubernetes
    +
VMs
    +
Physical Servers
    +
Applications
    +
Databases
    +
External systems
```

---

# 14. Example — Local Windows/Linux Server

Suppose you have:

```text
                    Kubernetes
                        │
                   Prometheus
                        │
                        │ scrape
                        ▼
                  192.168.1.50:9100
                        │
                  Node Exporter
                        │
                        ▼
                  Local Linux VM
```

Prometheus could collect:

```text
CPU
Memory
Disk
Network
Filesystem
Load
```

Then Grafana:

```text
                 Grafana
                    │
                    ▼
          ┌─────────────────────┐
          │ Local Server        │
          ├─────────────────────┤
          │ CPU          45%    │
          │ Memory       62%    │
          │ Disk         71%    │
          │ Network      12 MB/s│
          └─────────────────────┘
```

---

# 15. But Don't Do That Yet

I recommend **not adding the local server immediately**.

First, let's understand the Kubernetes monitoring stack we just installed.

Our learning sequence should be:

```text
Lesson 3
   │
   └── Install kube-prometheus-stack
            ↓
Lesson 4
   │
   └── Understand Prometheus Operator
            ↓
Lesson 5
   │
   └── Explore Prometheus targets
            ↓
Lesson 6
   │
   └── Explore Kubernetes metrics
            ↓
Lesson 7
   │
   └── PromQL
            ↓
Lesson 8
   │
   └── Grafana dashboards
            ↓
Lesson 9
   │
   └── Monitor external/local server
```

This will make the concepts much clearer.

---

# 🧪 Your Lesson 3 Practical Exercise

Let's do this **one step at a time**.

### Step 1

Run:

```bash
kubectl get nodes
```

### Step 2

Run:

```bash
helm version
```

### Step 3

Run:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

### Step 4

Run:

```bash
helm repo update
```

### Step 5

Create the namespace:

```bash
kubectl create namespace monitoring
```

### Step 6

Install:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring
```

### Step 7

Check:

```bash
kubectl get pods -n monitoring
```

---

**Stop there.**

Don't configure dashboards or alerts yet.

Send me the output of:

```bash
kubectl get pods -n monitoring
```

and we'll do the **next small step: understanding every pod that Helm installed and why it is there.**
