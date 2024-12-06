# **Update Strategies and Blue-Green Deployment in Kubernetes**

This guide explains the key Kubernetes update strategies, including **RollingUpdate**, **Recreate**, and **Blue-Green Deployment**, to manage application updates efficiently.

---

## **1. Kubernetes Update Strategies**

Kubernetes provides two main update strategies for Deployments:

### **1.1 RollingUpdate (Default)**

- **How it works:**  
  Updates Pods incrementally. A few Pods are updated at a time while the rest remain active to handle traffic.
- **Benefits:**
  - Minimal downtime.
  - Gradual rollout allows monitoring for issues.
- **Configuration Example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: my-registry/myapp:v2
        ports:
        - containerPort: 80
```
  - `maxUnavailable`: Maximum number of Pods that can be unavailable during the update.
  - `maxSurge`: Maximum number of additional Pods that can be created during the update.

**Commands:**
- Check rollout status:
  ```bash
  kubectl rollout status deployment myapp-deployment
  ```
- Rollback to the previous version:
  ```bash
  kubectl rollout undo deployment myapp-deployment
  ```

---

### **1.2 Recreate Strategy**

- **How it works:**  
  All existing Pods are terminated before new Pods are created.
- **Benefits:**
  - Suitable for applications that can't handle multiple versions simultaneously.
- **Configuration Example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: my-registry/myapp:v2
        ports:
        - containerPort: 80
```

**Commands:**
- Apply the updated Deployment:
  ```bash
  kubectl apply -f myapp-deployment.yaml
  ```

---

## **2. Blue-Green Deployment**

A Blue-Green Deployment enables you to deploy a new version of your application (Blue) alongside the current version (Green) and switch traffic between them seamlessly.

### **How it Works:**
1. **Green Environment:**  
   The current, stable version of the application serves all traffic.
2. **Blue Environment:**  
   The new version is deployed and tested without affecting users.
3. **Traffic Switch:**  
   Traffic is redirected from the Green environment to the Blue environment once testing is complete.

---

### **2.1 Green Deployment (Current Version)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp-container
        image: my-registry/myapp:v1
        ports:
        - containerPort: 80
```

---

### **2.2 Blue Deployment (New Version)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp-container
        image: my-registry/myapp:v2
        ports:
        - containerPort: 80
```

---

### **2.3 Manage Traffic with a Service**
Initially, the Service routes traffic to the Green Deployment:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: green
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

When ready to switch to the Blue Deployment, update the Service:
```bash
kubectl patch service myapp-service -p '{"spec":{"selector":{"app":"myapp","version":"blue"}}}'
```

---

### **2.4 Rollback**
If issues are detected with the Blue Deployment, redirect traffic back to the Green Deployment:
```bash
kubectl patch service myapp-service -p '{"spec":{"selector":{"app":"myapp","version":"green"}}}'
```

---

### **2.5 Advantages of Blue-Green Deployment**
- **Zero Downtime:** The traffic switch is seamless.
- **Quick Rollback:** Easy to revert to the Green environment if issues occur.
- **Testing in Production:** The Blue environment can be tested in a real-world scenario without affecting users.

---

## **3. Comparison of Strategies**

| **Strategy**      | **RollingUpdate**                | **Recreate**                  | **Blue-Green Deployment**         |
|--------------------|----------------------------------|--------------------------------|------------------------------------|
| **Downtime**       | Minimal                         | Yes                            | None                              |
| **Pod Availability** | At least some Pods are active. | None during updates.           | Both versions available.          |
| **Rollback**       | Incremental rollback.           | Requires redeployment.         | Instant traffic switch to Green.  |
| **Use Case**       | Standard production updates.    | State-conflicting applications.| Controlled, zero-downtime updates.|

