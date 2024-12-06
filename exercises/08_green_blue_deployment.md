# **Blue-Green Deployment with Frontend Application in Kubernetes**

This guide walks you through setting up and demonstrating **Blue-Green Deployment** for your frontend application in Kubernetes.

---

## **1. Prerequisites**

- A Kubernetes cluster (e.g., Minikube, K3s, or a cloud provider) is up and running.
- `kubectl` is installed and configured.
- The frontend manifest is ready.
- Namespace `workshop-basic-app` exists or will be created.

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

### **2.2. Deploy the Green (Current) Environment**

Create the **Green Deployment** (current version):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-green
  namespace: workshop-basic-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      version: green
  template:
    metadata:
      labels:
        app: frontend
        version: green
    spec:
      containers:
      - name: frontend
        image: ghcr.io/tom9eiger/frontend-demo:latest
        imagePullPolicy: Always
        env:
          - name: BACKEND_URL
            value: "http://flask-api-service.demo-cni-app.svc.cluster.local"
        ports:
        - containerPort: 80
```

Apply the Green Deployment:
```bash
kubectl apply -f frontend-green.yaml
```

Create a Service to route traffic to the Green environment:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: workshop-basic-app
spec:
  selector:
    app: frontend
    version: green
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Apply the Service:
```bash
kubectl apply -f frontend-service.yaml
```

---

### **2.3. Deploy the Blue (New) Environment**

Create the **Blue Deployment** (new version):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-blue
  namespace: workshop-basic-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      version: blue
  template:
    metadata:
      labels:
        app: frontend
        version: blue
    spec:
      containers:
      - name: frontend
        image: ghcr.io/tom9eiger/frontend-demo:v2
        imagePullPolicy: Always
        env:
          - name: BACKEND_URL
            value: "http://flask-api-service.demo-cni-app.svc.cluster.local"
        ports:
        - containerPort: 80
```

Apply the Blue Deployment:
```bash
kubectl apply -f frontend-blue.yaml
```

---

## **3. Traffic Management**

### **3.1. Initial Setup**

Initially, the `frontend-service` routes all traffic to the Green Deployment (`version: green`).

Verify the service:
```bash
kubectl get service frontend-service -n workshop-basic-app
```

---

### **3.2. Switch Traffic to Blue**

Update the Service to point to the Blue Deployment:
```bash
kubectl patch service frontend-service -n workshop-basic-app -p '{"spec":{"selector":{"app":"frontend","version":"blue"}}}'
```

Verify that the Service now points to Blue:
```bash
kubectl describe service frontend-service -n workshop-basic-app
```

---

### **3.3. Rollback to Green**

If any issues arise with the Blue Deployment, you can easily rollback by pointing the Service back to Green:
```bash
kubectl patch service frontend-service -n workshop-basic-app -p '{"spec":{"selector":{"app":"frontend","version":"green"}}}'
```

---

## **4. Testing Blue-Green Deployment**

1. **Access the Application:**
   Use a port-forward to access the frontend service:
   ```bash
   kubectl port-forward service/frontend-service -n workshop-basic-app 8080:80
   ```
   Open your browser or use `curl` to test:
   ```bash
   curl http://localhost:8080
   ```

2. **Test Traffic Switching:**
   - Initially, the response should indicate that traffic is handled by the Green Deployment.
   - After switching to Blue, the response should reflect the new version.

3. **Monitor Logs:**
   Check logs from both deployments:
   ```bash
   kubectl logs -l app=frontend,version=green -n workshop-basic-app
   kubectl logs -l app=frontend,version=blue -n workshop-basic-app
   ```

---

## **5. Cleanup**

Remove all resources:
```bash
kubectl delete deployment frontend-green -n workshop-basic-app
kubectl delete deployment frontend-blue -n workshop-basic-app
kubectl delete service frontend-service -n workshop-basic-app
kubectl delete namespace workshop-basic-app
```

---

## **6. Key Points**

- **Zero Downtime:** Blue-Green Deployment enables seamless traffic switching.
- **Quick Rollbacks:** Easily revert to the Green environment if issues occur.
- **Testing in Production:** Blue environment can be tested without impacting users.
