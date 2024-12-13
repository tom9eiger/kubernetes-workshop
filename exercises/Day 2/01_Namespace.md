
---

# **Namespace Workshop: Deploying and Testing Applications**

This workshop guides you through creating **dev** and **prod** namespaces in Kubernetes. You will deploy an **nginx** deployment and service along with a **busybox** Pod in each namespace to test DNS resolution.

---

## **1. Objectives**

- Learn how to create and manage Kubernetes namespaces.
- Deploy resources in separate namespaces.
- Test DNS resolution between Pods and Services within a namespace.

---

## **2. Prerequisites**

- Kubernetes cluster is up and running (e.g., Minikube or K3s).
- `kubectl` is installed and configured.

---

## **3. Steps**

### **Step 1: Create Namespaces**

1. Create `dev` and `prod` namespaces:
   ```bash
   kubectl create namespace dev
   kubectl create namespace prod
   ```

2. Verify the namespaces:
   ```bash
   kubectl get namespaces
   ```

---

### **Step 2: Deploy Applications in the `dev` Namespace**

1. **Create an nginx Deployment and Service in `dev`:**

   Save the following YAML as `nginx-dev.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
     namespace: dev
   spec:
     replicas: 2
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
           image: nginx
           ports:
           - containerPort: 80
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx
     namespace: dev
   spec:
     selector:
       app: nginx
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```

   Deploy it:
   ```bash
   kubectl apply -f nginx-dev.yaml
   ```

2. **Create a busybox Pod in `dev`:**

   Save the following YAML as `busybox-dev.yaml`:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: busybox
     namespace: dev
   spec:
     containers:
     - name: busybox
       image: busybox
       command: ["sleep", "3600"]
   ```

   Deploy it:
   ```bash
   kubectl apply -f busybox-dev.yaml
   ```

3. Verify all resources:
   ```bash
   kubectl get all -n dev
   ```

---

### **Step 3: Deploy Applications in the `prod` Namespace**

1. **Create an nginx Deployment and Service in `prod`:**

   Save the following YAML as `nginx-prod.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
     namespace: prod
   spec:
     replicas: 2
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
           image: nginx
           ports:
           - containerPort: 80
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx
     namespace: prod
   spec:
     selector:
       app: nginx
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```

   Deploy it:
   ```bash
   kubectl apply -f nginx-prod.yaml
   ```

2. **Create a busybox Pod in `prod`:**

   Save the following YAML as `busybox-prod.yaml`:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: busybox
     namespace: prod
   spec:
     containers:
     - name: busybox
       image: busybox
       command: ["sleep", "3600"]
   ```

   Deploy it:
   ```bash
   kubectl apply -f busybox-prod.yaml
   ```

3. Verify all resources:
   ```bash
   kubectl get all -n prod
   ```

---

### **Step 4: Test DNS Resolution**

1. **Access the `busybox` Pod in `dev`:**
   ```bash
   kubectl exec -it busybox -n dev -- /bin/sh
   ```

2. **Test connectivity to the nginx Service in `dev`:**
   ```bash
   nslookup nginx.dev.svc.cluster.local
   curl nginx
   ```

3. **Access the `busybox` Pod in `prod`:**
   ```bash
   kubectl exec -it busybox -n prod -- /bin/sh
   ```

4. **Test connectivity to the nginx Service in `prod`:**
   ```bash
   nslookup nginx.prod.svc.cluster.local
   curl nginx
   ```

---

## **5. Cleanup**

Delete all resources and namespaces:
```bash
kubectl delete namespace dev
kubectl delete namespace prod
```

---

## **6. Key Points**

- Kubernetes namespaces isolate resources logically.
- Services provide DNS names for accessing Deployments.
- Testing within namespaces ensures proper isolation and service discovery.
