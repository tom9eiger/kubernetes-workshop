# **Load Balancing Test in Kubernetes**

This guide helps you deploy your manifests to test load balancing between multiple replicas of an application in Kubernetes.

---

## **1. Prerequisites**

- Kubernetes cluster is up and running (e.g., Minikube, K3s, or cloud provider).
- `kubectl` is installed and configured to communicate with the cluster.
- The manifests for frontend and backend applications are ready.

---

## **2. Deployment Steps**

### **2.1. Deploy the Backend Application**

1. **Apply the Backend Deployment:**
   ```bash
   kubectl apply -f backend-deployment.yaml
   ```

2. **Apply the Backend Service:**
   ```bash
   kubectl apply -f backend-service.yaml
   ```

3. **Verify the Backend Deployment:**
   ```bash
   kubectl get pods -l app=backend
   ```
   Ensure the backend Pods are running.

4. **Verify the Backend Service:**
   ```bash
   kubectl get services backend-service
   ```
   Note the ClusterIP and port of the backend service.

---

### **2.2. Deploy the Frontend Application**

1. **Apply the Frontend Deployment:**
   ```bash
   kubectl apply -f frontend-deployment.yaml
   ```

2. **Apply the Frontend Service:**
   ```bash
   kubectl apply -f frontend-service.yaml
   ```

3. **Verify the Frontend Deployment:**
   ```bash
   kubectl get pods -l app=frontend
   ```
   Ensure the frontend Pods are running.

4. **Verify the Frontend Service:**
   ```bash
   kubectl get services frontend-service
   ```
   Note the ClusterIP and port of the frontend service.

---

## **3. Test Load Balancing**

### **3.1. Access the Frontend**

1. Use a port-forward to access the frontend service:
   ```bash
   kubectl port-forward service/frontend-service 8080:80
   ```

2. Open a browser or use `curl`:
   ```bash
   curl http://localhost:8080
   ```

3. Refresh multiple times to see responses from different backend Pods. The response will indicate which backend Pod handled the request.

---

## **4. Observe Load Balancing in Action**

1. **Check Backend Logs:**
   Observe which backend Pods are receiving requests:
   ```bash
   kubectl logs -l app=backend -f
   ```

2. **Simulate High Traffic:**
   Use a tool like `hey` or `wrk` to send multiple requests to the frontend:
   ```bash
   hey -z 10s -c 50 http://localhost:8080
   ```

3. Verify that the requests are distributed among the backend Pods by checking their logs.

---

## **5. Scaling for More Traffic**

1. **Scale the Backend Deployment:**
   ```bash
   kubectl scale deployment backend-deployment --replicas=5
   ```

2. **Scale the Frontend Deployment:**
   ```bash
   kubectl scale deployment frontend-deployment --replicas=3
   ```

3. Verify that additional Pods are running:
   ```bash
   kubectl get pods
   ```

4. Test load balancing again by sending traffic to the frontend service.

---

## **6. Cleanup**

To remove all resources:
```bash
kubectl delete -f backend-deployment.yaml
kubectl delete -f backend-service.yaml
kubectl delete -f frontend-deployment.yaml
kubectl delete -f frontend-service.yaml
```

---

## **Load Balancing Key Points**

- **Frontend Service:** Distributes traffic among frontend Pods.
- **Backend Service:** Distributes traffic among backend Pods.
- **Scaling:** Kubernetes automatically distributes traffic to new Pods when scaling.
- **Observability:** Logs and metrics can help confirm load balancing behavior.
