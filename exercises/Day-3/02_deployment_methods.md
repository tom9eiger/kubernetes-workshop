Here's a workshop to deploy your application using **Kustomize** and **Helm**, showcasing both methods.

# **Workshop: Deploying with Kustomize and Helm**

---

## **1. Prerequisites**

- **Minikube** or a Kubernetes cluster:
  ```bash
  minikube start
  ```
- **kubectl** and **Helm** are installed:
  ```bash
  kubectl version
  helm version
  ```

---

## **2. Application Overview**

The application consists of:
1. **Namespace**: Logical isolation for resources.
2. **Frontend**: A web UI deployed as a Deployment and exposed via a Service.
3. **Backend**: A backend service deployed as a Deployment and exposed via a Service.

File structure:
```
deployment-example-application/
├── app-namespace.yaml
├── backend-deployment.yaml
├── backend-service.yaml
├── frontend-deployment.yaml
├── frontend-service.yaml
```

---

## **3. Part 1: Deploy Using Kustomize**

### **Step 1: Prepare the Directory Structure**

Create the following structure for Kustomize:

```
kustomize/
├── base/
│   ├── app-namespace.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   └── kustomization.yaml
├── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patches.yaml
    ├── prod/
        ├── kustomization.yaml
        └── patches.yaml
```

### **Step 2: Create Base Configuration**

**base/kustomization.yaml**:
```yaml
resources:
  - app-namespace.yaml
  - backend-deployment.yaml
  - backend-service.yaml
  - frontend-deployment.yaml
  - frontend-service.yaml
```

### **Step 3: Create Overlays for Dev and Prod**

**overlays/dev/kustomization.yaml**:
```yaml
resources:
  - ../../base

patchesStrategicMerge:
  - patches.yaml
```

**overlays/dev/patches.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
```

**overlays/prod/kustomization.yaml**:
```yaml
resources:
  - ../../base

patchesStrategicMerge:
  - patches.yaml
```

**overlays/prod/patches.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
```

---

### **Step 4: Deploy Using Kustomize**

1. Deploy the **Dev** environment:
   ```bash
   kubectl apply -k kustomize/overlays/dev
   ```
2. Verify the resources:
   ```bash
   kubectl get all -n example-app
   ```
3. Deploy the **Prod** environment:
   ```bash
   kubectl apply -k kustomize/overlays/prod
   ```

---

## **4. Part 2: Deploy Using Helm**

### **Step 1: Create a Helm Chart**

Generate a Helm chart:
```bash
helm create example-app
```

This creates the following structure:
```
example-app/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── namespace.yaml
```

---

### **Step 2: Customize the Chart**

1. **Namespace**: Define in `templates/namespace.yaml`:
   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: {{ .Values.namespace }}
   ```
   Add `namespace` to `values.yaml`:
   ```yaml
   namespace: example-app
   ```

2. **Backend**: Add the following in `templates/deployment.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: backend
     namespace: {{ .Values.namespace }}
   spec:
     replicas: {{ .Values.backend.replicas }}
     selector:
       matchLabels:
         app: backend
     template:
       metadata:
         labels:
           app: backend
       spec:
         containers:
         - name: backend
           image: {{ .Values.backend.image }}
           ports:
           - containerPort: {{ .Values.backend.port }}
   ```

3. **Frontend**: Similarly customize `deployment.yaml` for the frontend.

4. **Service**: Add a template for services in `templates/service.yaml`.

---

### **Step 3: Customize `values.yaml`**

Define values:
```yaml
namespace: example-app

backend:
  replicas: 2
  image: backend-image:latest
  port: 8080

frontend:
  replicas: 2
  image: frontend-image:latest
  port: 80
```

---

### **Step 4: Deploy Using Helm**

1. Install the Helm chart:
   ```bash
   helm install example-app ./example-app
   ```
2. Verify the resources:
   ```bash
   kubectl get all -n example-app
   ```
3. Upgrade the release to scale the backend:
   ```bash
   helm upgrade example-app ./example-app --set backend.replicas=5
   ```

---

## **5. Cleanup**

To remove resources:

**Kustomize**:
```bash
kubectl delete -k kustomize/overlays/dev
kubectl delete -k kustomize/overlays/prod
```

**Helm**:
```bash
helm uninstall example-app
```

---

## **6. Key Takeaways**

- **Kustomize**:
  - Great for managing overlays (dev, prod) with minimal duplication.
  - Simple for YAML-focused workflows.
- **Helm**:
  - Powerful for reusable and version-controlled deployments.
  - Ideal for complex applications with many customizations.
