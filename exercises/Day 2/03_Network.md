Here’s the **README.md** with the `iptables` analysis from your original content added to the previous structure:

---

# **Kubernetes Workshop: Deploying Applications with Network Policies**

This workshop demonstrates the deployment of a **Flask API backend** and a **frontend application** in Kubernetes. You'll also explore **kube-proxy**, **CNI**, and **Network Policies** while analyzing `iptables` rules for Service traffic.

---

## **Setup Minikube**

### **1. Start Minikube with Calico**

Start Minikube with 1 Control Plane Node and 3 Worker Nodes, using the **Calico CNI plugin**:

```bash
minikube start --network-plugin=cni --cni=calico --nodes=4
```

Verify that Calico is running:

```bash
kubectl get pods -n kube-system | grep calico
```

---

### **2. Install MetalLB**

Apply the MetalLB manifests:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
```

---

### **3. Configure MetalLB**

Determine the IP address range that your Minikube cluster uses:

```bash
minikube ip
```

Assume the Minikube IP is `192.168.49.2`. Use an IP range in the same subnet (e.g., `192.168.49.30-192.168.49.40`). Create a MetalLB configuration file:

```yaml
# metallb-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.49.30-192.168.49.40
```

Apply the configuration:

```bash
kubectl apply -f metallb-config.yaml
```

---

## **Deploy to Kubernetes**

### **1. Apply Namespace**

```bash
kubectl create namespace demo-cni-app
```

---

### **2. Deploy Backend Application**

```yaml
# backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: demo-cni-app
spec:
  replicas: 2
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
        image: ghcr.io/tom9eiger/backend-demo:latest
        ports:
        - containerPort: 5000
```

```yaml
# backend-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: demo-cni-app
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
  type: ClusterIP
```

Deploy:

```bash
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
```

---

### **3. Deploy Frontend Application**

```yaml
# frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: demo-cni-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: ghcr.io/tom9eiger/frontend-demo:latest
        ports:
        - containerPort: 80
```

```yaml
# frontend-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: demo-cni-app
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

Deploy:

```bash
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```

---

## **Analyze kube-proxy and iptables**

### **1. Get ClusterIP and Endpoints**

Check the ClusterIP and endpoints for your services:

```bash
kubectl get svc -n demo-cni-app
kubectl get endpoints -n demo-cni-app
```

---

### **2. Inspect `iptables` NAT Rules**

1. SSH into the Minikube node:

   ```bash
   minikube ssh
   ```

2. List the NAT rules for `KUBE-SERVICES`:

   ```bash
   sudo iptables -t nat -L KUBE-SERVICES
   ```

   Example output:
   ```bash
   Chain KUBE-SERVICES (2 references)
   target     prot opt source               destination         
   KUBE-SVC-6YNYFUIKGNIA7RFX  tcp  --  anywhere             10.108.198.28        /* demo-cni-app/backend cluster IP */ tcp dpt:5000
   ```

3. Check the NAT rules for a specific service:

   ```bash
   sudo iptables -t nat -L KUBE-SVC-6YNYFUIKGNIA7RFX
   ```

   Example output:
   ```bash
   Chain KUBE-SVC-6YNYFUIKGNIA7RFX (1 references)
   target     prot opt source               destination         
   KUBE-MARK-MASQ  tcp  -- !10.244.0.0/16        10.108.198.28        /* demo-cni-app/backend cluster IP */ tcp dpt:5000
   KUBE-SEP-J7YQFRES3OILODCJ  all  --  anywhere             anywhere             /* demo-cni-app/backend -> 10.244.0.3:5000 */
   ```

4. Inspect the rules for the service endpoint:

   ```bash
   sudo iptables -t nat -L KUBE-SEP-J7YQFRES3OILODCJ
   ```

   Example output:
   ```bash
   Chain KUBE-SEP-J7YQFRES3OILODCJ (1 references)
   target     prot opt source               destination         
   KUBE-MARK-MASQ  all  --  10.244.0.3           anywhere             /* demo-cni-app/backend */
   DNAT       tcp  --  anywhere             anywhere             /* demo-cni-app/backend */ tcp to:10.244.0.3:5000
   ```

5. Exit Minikube:

   ```bash
   exit
   ```

---

## **Demo Network Policies**

### **1. Test Without Network Policy**

Deploy a debug pod:

```yaml
# debug-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
  namespace: demo-cni-app
spec:
  containers:
  - name: debug-container
    image: busybox
    command: ["sleep", "3600"]
```

```bash
kubectl apply -f debug-pod.yaml
```

Test connectivity:

```bash
kubectl exec -it debug-pod -n demo-cni-app -- wget -qO- http://backend.demo-cni-app.svc.cluster.local:5000/api
```

---

### **2. Apply Network Policy**

Restrict traffic to the backend service:

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: demo-cni-app
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 5000
```

```bash
kubectl apply -f network-policy.yaml
```

Re-test connectivity:

```bash
kubectl exec -it debug-pod -n demo-cni-app -- wget -qO- http://backend.demo-cni-app.svc.cluster.local:5000/api
```

---

## **Cleanup**

```bash
kubectl delete -f backend-deployment.yaml
kubectl delete -f backend-service.yaml
kubectl delete -f frontend-deployment.yaml
kubectl delete -f frontend-service.yaml
kubectl delete -f network-policy.yaml
kubectl delete -f debug-pod.yaml
kubectl delete namespace demo-cni-app
```

