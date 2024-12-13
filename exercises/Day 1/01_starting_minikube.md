---

# **Minikube Setup for Kubernetes Exercises**

This guide will help you set up a **3-node Minikube cluster** and provide an introduction to basic Minikube CLI commands.

---

## **1. Starting Minikube with 3 Nodes**

Minikube allows you to simulate a multi-node Kubernetes cluster locally. Follow these steps:

### **Start Minikube**
```bash
minikube start --nodes=3 --driver=docker
```

- `--nodes=3`: Creates 1 control plane node and 2 worker nodes.
- `--driver=docker`: Uses Docker as the backend for running Minikube.

### **Check the Cluster Status**
```bash
kubectl get nodes
```
You should see three nodes:  
- 1 Control Plane (e.g., `minikube`)  
- 2 Workers (e.g., `minikube-m02`, `minikube-m03`)

---

## **2. Useful Minikube CLI Commands**

### **Cluster Management**
- **Start a cluster**:
  ```bash
  minikube start
  ```
- **Stop a cluster**:
  ```bash
  minikube stop
  ```
- **Delete a cluster**:
  ```bash
  minikube delete
  ```

### **Cluster Information**
- **Get the status of the cluster**:
  ```bash
  minikube status
  ```
- **List running clusters**:
  ```bash
  minikube profile list
  ```

### **Access Applications**
- **Get the URL of a service**:
  ```bash
  minikube service <service-name>
  ```
- **Enable Minikube addons** (e.g., Ingress):
  ```bash
  minikube addons enable ingress
  ```

### **Advanced**
- **SSH into a node**:
  ```bash
  minikube ssh --node=minikube-m02
  ```
- **View resource usage (CPU, memory, etc.)**:
  ```bash
  minikube dashboard
  ```

---

## **3. Cleaning Up**

When you finish the exercises, you can stop and delete the cluster to free resources:
```bash
minikube stop
minikube delete
```
