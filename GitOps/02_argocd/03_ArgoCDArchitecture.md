## 🧩 Argo CD Architecture (Explained Clearly)

Argo CD follows a **GitOps architecture** where Git is the *single source of truth* and your Kubernetes cluster is continuously synced to it.

---

# 🏗️ Core Architecture Overview

![Image](https://images.openai.com/static-rsc-4/ge98qTWgX6VbrA8pQDkHd4IuxvtjqMoWnpyYPb9eoE_KMd9r2WBanEwAGNbrdeLK1fWx26JwBCSQ0lAq5T2g0olP8rzNwjklQZLzhs59NB8ygJZWQb3sEk-WG8Q2RZabiZ8GxyjUwIEKR_OAFIiF5S4buyBPPBn3guPyzV1AK6ys2x5IPuYIf4gLcrhbMgzP?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/XsrktILCndBoFgUHxV1MIhNg9b0-lYi_ivADKk8bzxmiCCfbI1dnva-Xgj6He9T3xXM4LjXLL300fuVbiGRmUJdvUJ1yDuGyFpBUxsHg5nBXdD2dz-p-iHy7Hfat1ERtXS9I4EYfXu2cbgSnS-40GXVxvrXeef7jgyQyh7HyeYoRZ4s_TSQtUsrYW9w7N7xO?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/qDyZuKawvsX7tmw3NJKmF6zfr5OrPPSM-c_tUvSuSLg0smsjISqI81WShVWgEAfK-eluaE06N-Z8R2QTYXyu5ZCHPeHlIkztztdLxlIjfAHkiHoo1ZVTdiOV1BjiNee6cbPPBg4-NLSR-_7vnXDe0Wsx9m48HscHkv8isHLldXlQ32hVRLPeMpWYB5-TY7Yd?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/dw38SWYdhDyUBIDKwVHhOAaZ_ocv3RfdX-0diZD9eA4kdqgjvGnHv21vkPiOFSYEnOa7namGE7C_7sbOvxP3de9mBUEQPG5aJlTcZX3H_Zhrm9G7sOICnHNY9esxPztHZ59rBHT8LLMMv5DUB6ELvsznoQYDXM-k0BKAhMAo8FTDaiFI6uJqI5ao4XPL5QEm?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/E9x8SEj-ftok6lyTWMewI1R8YpieRGRtVlDV-14J719BwkdjYs-2bhyionRn48_CH8tBYZhdJ4IBqsVC8uKraa5dag2YQnjiW8CjsNnTcvIQ_pON4tDE5rPIinTAFaA-DYLq0zWH9QNetF7xQ0vULZBV2XDoeCT_TcaS4FtJYsHErFQH8gvf0KPXXoozeKrx?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/wh4zR6s_0UYwOhtGI-fZFnBwcRpNWWrYixWbIA-bnXMd00aDF9Aewu0pOUaIpsb17-leLpGA0cK_B7N78VUiR6X4M7ayrAlZD8-_5sArwolMlFX3xHHPWH72PU9Qu3fG7KUTmJBe6ZWFRDggtujZh69bv2HLw5RNDC7WsseruB8vHoGh_j11grYsgxo_4va8?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/LsAtv32585nDHbmdIHy5vXt3GkEBF3Ysa_gsXuIUMzPHm8xT_IfuiqcbKGhyLsbMaRyRpEfrj1w3GsygLvbA6ejNs_xrhbfv8NqLXO1_Y1q6_MTgIWDh59mhuLO5KH1labnQKcz4VMef22H6Jg0MMrg-4sAcM9eIRW_HxDKSis_XJY7_LZCoJm6oYo9fasdz?purpose=fullsize)

---

# 🔑 Core Components

## 1. 🖥️ API Server (`argocd-server`)

* Entry point for:

  * UI (Web dashboard)
  * CLI (`argocd`)
  * REST API
* Handles authentication (SSO, tokens)
* Talks to other internal components

👉 Think of it as the **control panel**

---

## 2. ⚙️ Application Controller (`argocd-application-controller`)

* The **brain of Argo CD**
* Continuously compares:

  * Desired state (Git)
  * Live state (cluster)
* Performs:

  * Sync
  * Self-healing
  * Drift detection

👉 This is where GitOps actually happens

---

## 3. 📦 Repo Server (`argocd-repo-server`)

* Pulls manifests from Git repos
* Supports:

  * Helm charts
  * Kustomize
  * Plain YAML
* Generates Kubernetes manifests

👉 Converts Git → Kubernetes manifests

---

## 4. 🔐 Redis (`argocd-redis`)

* Caches:

  * Application state
  * Git data
* Improves performance

---

## 5. 🧠 Dex (Optional)

* Identity provider for SSO
* Integrates with:

  * GitHub
  * LDAP
  * OIDC

---

# 🔄 How It Works (Flow)

## Step-by-step:

1. You push code to Git
2. Argo CD detects change
3. Repo Server pulls & renders manifests
4. Application Controller compares states
5. If drift found → syncs cluster
6. Kubernetes updated automatically

---

# 🔁 GitOps Workflow

![Image](https://images.openai.com/static-rsc-4/WzeJiyJMLOQ0d4LfxzzW7w3k5itFJ2mmS3TrqRqQmvVEPF7atPgOqyjuPzxa6xr3On-0kEvxM1ilKBPnTTXxG3jjT1h6KBomUgO8DL0FL4AIAVrDUF-T3rNOwosmovFO1ZUP8KdXkF1nmQaV1Kx-XKfbKlkFLH8yxC9TA-k6Z7nswkGD_mxqweZQ_jS2wwUl?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/Eo1aNuX3BWMfRlS4UaKMGzZJ806RNrjt2oskHIYbqkK-ricC_yrouQA4x5a1l0egOLfV7-nc12nU859wO9iL_PGoI1QruTp_C5v3yl-sMxCMlOwvxjZphC8Fa0Jo22-UVOWkY1E8SuQ0JILyo9AKrENdjgEu2fVLciGGdI0s7vJgHbUZx7eZpABpKKpzHFet?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/vebENk5zk2Si4uVqbgJKzwfXMKnM2OjW9LJ4GRbiaiXuK34cB_-JnCr_y6CZTrAlN2s6jqdVtljw2-cSq90sLnLDMFVXlxmZziLPx-P3q5fS1VkS0-5SFO2CMqx3GvWevHUuM728ZuyFrZid7wiNu4f9dX0ef9T0dqwn8Js_YK0uj9sXGUj4VG1txySVm831?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/9swTl6P2i1jQVyaXFOy2mtSS9rFVNkDi3MIBOLHwRK_ZlQRsCBqcSj42KbNdkK6Lt7Dkq1RChWs3Dq_FTnuePi9Uw_D8KypnZEL-TFxztE2HyrXP4UJnyhVmcZu_9_Th5eCF3XCAYKhO9RbVy000mr91C4DAkAe-sHfPsBJAglXOjdI9Hp-ldaBUIICq6Cb5?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/00BLF3_GNFV4i4qN3YFWdDf1r5970_teYFzrKk3c0gQEaV8DpDUzCka3_gT6LDdaz8hoFM0jigbq-j4uIEbHMQxY9nbGgil4dCveTWr-3_piIkI5aZwP6QrpchkNncsdDPkXacqXas8BWoh3w_ScOO-oZcxJvAB0wkdeweSdBE9eL9kGzPjOD72TMsJzYLfj?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/9o4z21SAO2A3NUDqtU9DxbQxOWS1_pNeEs6EKxaGZ8R1SLCRj0_HrFNuYarwnr1770l55lkIYbulEb3kG1Sap8sNN882SxY3Dagi4uKcZMyx2xj9YoL46GVcjxuSREYs60pyjODNlEwCS-7lF3NoEwEDb-_5CLg7QjxiRFoWsdGB-Qkf6teFaV2q3X8dSZnw?purpose=fullsize)

---

# 📦 Key Concepts

## 📁 Application

* A mapping:

  * Git repo
  * Path
  * Cluster + namespace

---

## 🔄 Sync

* Applies desired state → cluster

Types:

* Manual sync
* Auto sync

---

## 🔍 Drift Detection

* Detects differences between:

  * Git
  * Live cluster

---

## 🛠️ Self-Healing

* Automatically fixes changes made manually in cluster

---

# 🔐 Security Model

* RBAC controls access
* Works with Kubernetes RBAC
* Supports SSO via Dex

---

# ⚡ High-Level Summary

| Component   | Role               |
| ----------- | ------------------ |
| API Server  | UI + API           |
| Controller  | Sync engine        |
| Repo Server | Manifest generator |
| Redis       | Cache              |
| Dex         | Authentication     |

---

# 🧠 Simple Analogy

Think of Argo CD like this:

* **Git repo** → Blueprint 📘
* **Controller** → Inspector 👀
* **Kubernetes cluster** → Building 🏗️
* **Sync** → Construction work 🔨

If the building doesn't match the blueprint → Argo CD fixes it.

---

# 🚀 Real-World Add-ons

In production, you often add:

* Ingress (NGINX / Istio)
* External DB (instead of Redis)
* HA setup (multiple controllers)

---


