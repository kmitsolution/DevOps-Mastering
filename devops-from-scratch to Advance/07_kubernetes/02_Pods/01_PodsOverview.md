# Kubernetes Pods – Creation and Management

## 1. Introduction

In Kubernetes, a **Pod** is the **smallest deployable unit**.
A pod represents a running process in a cluster and contains **one or more containers**.

Most commonly, a pod runs **a single container**, but it can also run **multiple containers that share resources**.

Containers inside a pod share:

* Network namespace
* Storage volumes
* IP address
* localhost communication

Example structure:

```
Pod
 └── Container (nginx)
```

or

```
Pod
 ├── Container 1 (app)
 └── Container 2 (log collector)
```

---

# 2. Verify Kubernetes Cluster

Before creating pods, verify that the Kubernetes cluster is working correctly.

### Check Nodes

```bash
kubectl get nodes
```

Expected output:

```
NAME      STATUS   ROLES
kmaster   Ready    control-plane
worker1   Ready    <none>
worker2   Ready    <none>
```

---

### Check Namespaces

```bash
kubectl get ns
```

---

### Verify Flannel Network Pods

```bash
kubectl get pods -n kube-flannel
```

---

### Check All Running Pods

```bash
kubectl get pods
```

---

# 3. Methods to Create Pods

Pods can be created using **two methods**:

1️⃣ **Using Kubernetes commands**
2️⃣ **Using Manifest files (YAML files)**

---

# 4. Creating Pods Using Kubernetes Commands

The simplest way to create a pod is using the **kubectl run command**.

### Check kubectl run help

```bash
kubectl run --help
```

---

### Create a Pod with Nginx

```bash
kubectl run nginx-pod --image nginx
```

---

### Verify Pod Creation

```bash
kubectl get pods
```

Example output:

```
NAME        READY   STATUS    AGE
nginx-pod   1/1     Running   10s
```

---

# 5. Pod Lifecycle

When a pod is created, it goes through several states:

```
Pending → ContainerCreating → Running
```

Possible states:

* Pending
* Running
* Failed
* Succeeded

---

# 6. View Pod Details

To see complete configuration of a pod:

### YAML Format

```bash
kubectl get pods nginx-pod -o yaml
```

---

### JSON Format

```bash
kubectl get pods nginx-pod -o json
```

---

# 7. Edit Pod at Runtime

You can modify a pod configuration using:

```bash
kubectl edit pod nginx-pod
```

This opens the pod configuration in the default editor.

---

# 8. Check Where Pod is Running

To see which node the pod is running on:

```bash
kubectl get pods -o wide
```

Example output:

```
nginx-pod   1/1 Running   worker1
```

---

# 9. Access a Pod

To execute commands inside a container:

### Open interactive shell

```bash
kubectl exec -it nginx-pod -- bash
```

Exit the container:

```
exit
```

---

### Execute command without entering container

```bash
kubectl exec -it nginx-pod -- ls
```

---

# 10. Pod Logs

To check container logs:

```bash
kubectl logs nginx-pod
```

or

```bash
kubectl logs nginx-pod -n default
```

---

# 11. Pod Debugging

To view detailed information about a pod:

```bash
kubectl describe pod nginx-pod
```

This command shows:

* Pod events
* Container status
* Image used
* Node information

---

# 12. Change Container Image in a Pod

Check container image:

```bash
kubectl get pod nginx-pod -o yaml | grep -i image
```

Update image:

```bash
kubectl set image pod nginx-pod nginx-pod=tomcat
```

After updating the image, Kubernetes will:

* Remove the old container
* Create a new container

The pod name remains the same.

---

# 13. Test Pod Application

Check pod IP:

```bash
kubectl get pods -o wide
```

Example:

```
nginx-pod   10.244.1.12
```

Test application:

```bash
curl 10.244.1.12
```

---

# 14. Delete a Pod

To delete a pod:

```bash
kubectl delete pod nginx-pod
```

---

# 15. Creating Pods Using Manifest Files

Pods can also be created using **YAML configuration files**.

Manifest files describe Kubernetes resources declaratively.

Advantages:

* Version control
* Easy updates
* Infrastructure as code

---

# 16. Create Manifest Directory

```bash
mkdir manifest
cd manifest
```

---

# 17. Example Pod Manifest File

### pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web-app
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

---

# 18. Create Pod Using Manifest

```bash
kubectl create -f pod.yaml
```

Verify:

```bash
kubectl get pods
```

---

# 19. Update Pod Using Manifest

```bash
kubectl apply -f pod.yaml
```

---

# 20. Delete Pod Using Manifest

```bash
kubectl delete -f pod.yaml
```

---

# 21. Generate Manifest File Using kubectl

Kubernetes can generate YAML files automatically.

Example:

```bash
kubectl run pod1 --image nginx --dry-run=client -o yaml
```

---

### Save YAML File

```bash
kubectl run pod1 --image nginx --dry-run=client -o yaml > pod1.yaml
```

---

# 22. Example Generated Manifest

### pod1.yaml

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: nginx
    name: pod1
```

---

# 23. Create Pod from YAML

```bash
kubectl create -f pod1.yaml
```

Verify:

```bash
kubectl get pods
```

---

# 24. Example Custom Pod

### myapp-pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: myapp
spec:
  containers:
  - name: myapp
    image: nginx
```

Create pod:

```bash
kubectl create -f myapp-pod.yaml
```

---

# 25. Another Example Pod

### newpod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: new-pod
  labels:
    app: nginx-app
spec:
  containers:
  - name: c1
    image: nginx
```

Create pod:

```bash
kubectl create -f newpod.yaml
```

---

# 26. Verify Pods

```bash
kubectl get pods
```

---

# 27. Delete Pods

```bash
kubectl delete -f myapp-pod.yaml
```

or

```bash
kubectl delete pod new-pod
```

---

# 28. Methods to Create Manifest Files

Manifest files can be created using:

1️⃣ **Kubernetes Documentation**
[https://kubernetes.io/docs](https://kubernetes.io/docs)

2️⃣ **Visual Studio Code (YAML editing)**

3️⃣ **kubectl command**

```
--dry-run=client -o yaml
```

Example:

```
kubectl run pod1 --image nginx --dry-run=client -o yaml
```

---

# 29. Summary

In this session we learned:

* What a **Kubernetes Pod** is
* Pod lifecycle
* Creating pods using **kubectl commands**
* Accessing containers using **kubectl exec**
* Viewing logs and debugging pods
* Updating container images
* Creating pods using **YAML manifest files**
* Generating YAML files using **--dry-run=client**

