## 🧩 Argo CD Application CRD Architecture

In Argo CD, the **Application CRD** is the *core object* that defines how your app is deployed from Git to Kubernetes.

---

# 🏗️ High-Level Architecture

![Image](https://images.openai.com/static-rsc-4/XsrktILCndBoFgUHxV1MIhNg9b0-lYi_ivADKk8bzxmiCCfbI1dnva-Xgj6He9T3xXM4LjXLL300fuVbiGRmUJdvUJ1yDuGyFpBUxsHg5nBXdD2dz-p-iHy7Hfat1ERtXS9I4EYfXu2cbgSnS-40GXVxvrXeef7jgyQyh7HyeYoRZ4s_TSQtUsrYW9w7N7xO?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/Dum7dP7YM2_BNUMRxm3e1S5PuB5Wp413Q78fMd3SNR0fnflINXz5wmvk9dsFDRL83rnzTqm57yX6j0DQD7LrneDB9QpOe35oWUx2hW0TcewCNdacBIxKyQHQM0HlmKO1WExG7-VlBJr41CZZF13NdTRkmNwni6K4DdLyfoCtCU1eYPzFxt-SHDVwo9Fu6F5M?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/hSiNngZvQlzHClG_cigFdaeURZk23J-UA1vExb4KTPrhDJnt7JEVLtl_WtWBJSff0zg9iAAfN_1DRwr54P5CBBopDc0ylQ9TNnLCijCP8o_07ggWBG8ZlC44ie1hmCzzxsLLhb4Of0s99jnAeg-iaMYLoFd6cSeE-zmFpOxw4duyd4uzuJFIb7VONNfaVlu7?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/lYVDray-mSDieSNyHV2jp1tE4_UAag1D0PkZ59YxnhdbNhD56vJU4lRLeRWNQzntXJizIjgaXAxUT5h9btSP827AOw__oCs7Az79nywYL2w_x9klWCPpOYb76Zfo4Czj89JIbMMeemJfrARn0_epOfjunkAzjq7o_5Noun4pE0euQrPQQ6hzKSRgs36yNd-1?purpose=fullsize)

---

# 🔑 What is Application CRD?

* A **Custom Resource Definition (CRD)** in Kubernetes
* Kind: `Application`
* Defines:

  * **Source** (Git repo / Helm / Kustomize)
  * **Destination** (cluster + namespace)
  * **Sync behavior**

👉 It acts as the **bridge between Git and the cluster**

---

# 📦 Application CRD Structure

Here’s a simplified YAML:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  source:
    repoURL: https://github.com/example/repo.git
    path: manifests/
    targetRevision: HEAD

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  project: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

# 🧠 Core Architecture Components (with CRD)

## 1. 📁 Application (CRD Object)

* Stored in Kubernetes API
* Managed like any other K8s resource
* Declarative definition of app

---

## 2. ⚙️ Application Controller

* Watches Application CRDs
* Runs reconciliation loop:

  * Desired state (Git)
  * Live state (cluster)

---

## 3. 📦 Repo Server

* Reads `spec.source`
* Fetches repo
* Renders:

  * Helm → templates
  * Kustomize → overlays
  * YAML → direct

---

## 4. 🎯 Kubernetes API

* Stores live resources
* Controller applies manifests here

---

# 🔄 Reconciliation Flow

## Step-by-step:

1. Application CRD created
2. Controller detects new app
3. Repo Server pulls manifests
4. Controller compares:

   * Git vs Live
5. If mismatch:

   * Sync triggered
6. Resources applied to cluster

---

# 🔁 Flow Diagram (Conceptual)

```
Application CRD
      ↓
Application Controller
      ↓
Repo Server (fetch + render)
      ↓
Compare (desired vs live)
      ↓
kubectl apply (sync)
      ↓
Kubernetes cluster updated
```

---

# 📌 Key Fields Explained

## 🔹 source

* Git repo, Helm chart, or Kustomize path
* Defines *what to deploy*

---

## 🔹 destination

* Cluster API server
* Namespace
* Defines *where to deploy*

---

## 🔹 syncPolicy

Controls automation:

* `automated` → auto deploy
* `prune` → delete removed resources
* `selfHeal` → fix drift

---

## 🔹 project

* Logical grouping + RBAC control
* Multi-team isolation

---

# 🔍 Advanced Concepts

## 🧪 Multi-Source Apps

One Application can use:

* Helm + values
* Multiple repos

---

## 🔁 Sync Strategies

* Manual
* Auto
* Sync waves (ordering resources)

---

## 🧹 Pruning

* Deletes resources removed from Git

---

## 🧠 Drift Detection

* Detects manual changes in cluster

---

# ⚡ Why Application CRD is Powerful

* Fully **declarative**
* Stored in Git → true GitOps
* Kubernetes-native object
* Enables automation & self-healing

---

# 🧠 Simple Analogy

* **Application CRD** = Deployment plan 📄
* **Git repo** = Source blueprint 📘
* **Controller** = Supervisor 👷
* **Cluster** = Construction site 🏗️

---

# ✅ Summary

| Piece           | Role                  |
| --------------- | --------------------- |
| Application CRD | Defines app           |
| Controller      | Sync engine           |
| Repo Server     | Manifest generator    |
| Kubernetes      | Execution environment |

---


