# **Frontend Deployment with Update Strategy in Kubernetes**

This guide walks you through deploying the frontend application, scaling it, and demonstrating update strategies (RollingUpdate and Recreate) in Kubernetes.

---

## **1. Prerequisites**

- A Kubernetes cluster is up and running (e.g., Minikube, K3s, or a cloud provider).
- `kubectl` is installed and configured.
- The frontend manifest is ready.

---

## **2. Deployment Steps**

### **2.1. Create the Namespace**

Ensure the namespace `workshop-basic-app` exists:
```bash
kubectl create namespace workshop-basic-app
```

Verify the namespace:
```bash
kubectl get namespaces
```

---

### **2.2. Deploy the Frontend Application**

Apply the provided frontend manifest:
```bash
kubectl apply -f frontend-deployment.yaml
```

Verify the deployment:
```bash
kubectl get deployments -n workshop-basic-app
kubectl get pods -n workshop-basic-app
```

---

### **3. Scaling the Deployment**

Scale the frontend Deployment to handle more traffic:
```bash
kubectl scale deployment frontend --replicas=3 -n workshop-basic-app
```

Verify the scaling:
```bash
kubectl get pods -n workshop-basic-app
```

---

## **4. Update Strategies**

### **4.1. RollingUpdate (Default)**

- **Behavior:** Updates Pods incrementally, ensuring at least some Pods remain available during the update process.
- **Benefits:** Minimal downtime; suitable for most production environments.

#### **Demonstrate RollingUpdate**

1. Update the `image` in the manifest:
   ```yaml
   containers:
   - name: frontend
     image: ghcr.io/tom9eiger/frontend-demo:v2
   ```
2. Apply the updated manifest:
   ```bash
   kubectl apply -f frontend-deployment.yaml
   ```
3. Observe the update process:
   ```bash
   kubectl rollout status deployment frontend -n workshop-basic-app
   ```
4. Verify that the new Pods are running:
   ```bash
   kubectl get pods -n workshop-basic-app
   ```

---

### **4.2. Recreate Strategy**

- **Behavior:** Terminates all existing Pods before creating new Pods.
- **Benefits:** Ensures no overlap between old and new versions; suitable for stateful applications or significant configuration changes.

#### **Modify the Update Strategy**

Update the `strategy` in the Deployment manifest:
```yaml
spec:
  strategy:
    type: Recreate
```

Apply the updated manifest:
```bash
kubectl apply -f frontend-deployment.yaml
```

Observe the update process:
```bash
kubectl rollout status deployment frontend -n workshop-basic-app
```

---

## **5. Monitoring and Debugging**

### **Check Logs**
View logs for a specific Pod:
```bash
kubectl logs <pod-name> -n workshop-basic-app
```

### **Monitor Events**
Observe events during the update:
```bash
kubectl get events -n workshop-basic-app --sort-by='.metadata.creationTimestamp'
```

### **Inspect the Deployment**
Get detailed information about the Deployment:
```bash
kubectl describe deployment frontend -n workshop-basic-app
```

---

## **6. Cleanup**

Remove the Deployment and namespace:
```bash
kubectl delete deployment frontend -n workshop-basic-app
kubectl delete namespace workshop-basic-app
```

---

## **7. Key Points**

- **RollingUpdate:** Incremental updates; minimal downtime.
- **Recreate:** Full teardown before deployment; causes downtime.
- Use `kubectl rollout` commands to monitor and manage updates.
