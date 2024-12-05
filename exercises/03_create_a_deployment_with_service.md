# **Create a Deployment and Expose it with a NodePort Service**

This guide explains how to deploy an application using a Deployment and expose it for external access using a NodePort Service.

---

## **1. Step 1: Define the Deployment**

Create a YAML file for the Deployment (`nginx-deployment.yaml`):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

Apply the Deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

---

## **2. Step 2: Verify the Deployment**

### **Check the Pods**
```bash
kubectl get pods
```

### **Describe the Deployment**
```bash
kubectl describe deployment nginx-deployment
```

---

## **3. Step 3: Define the Service**

Create a YAML file for the Service (`nginx-service.yaml`):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

Apply the Service:

```bash
kubectl apply -f nginx-service.yaml
```

---

## **4. Step 4: Verify the Service**

### **Check the Service**
```bash
kubectl get services
```

Look for the assigned **NodePort** in the `PORT(S)` column (e.g., `80:32000/TCP`).

---

## **5. Step 5: Access the Service**

### **Retrieve the Service URL using Minikube**
```bash
minikube service nginx-service --url
```

### **Access the Application**
Open the provided URL in your browser to access the NGINX application.

---

## **6. Cleanup (Optional)**

To remove the Deployment and Service:

```bash
kubectl delete deployment nginx-deployment
kubectl delete service nginx-service
```
