# **Exploring Kubernetes with kubectl**

This guide demonstrates advanced `kubectl` commands to interact with Kubernetes resources, gather detailed information, and modify configurations.

---

## **1. Basic Resource Information**

### **List All Resources**
```bash
kubectl get all
```
This lists all Pods, Deployments, Services, ReplicaSets, and more in the current namespace.

### **Get Resources in All Namespaces**
```bash
kubectl get all --all-namespaces
```

### **Output Resources as YAML**
```bash
kubectl get pod <pod-name> -o yaml
```

---

## **2. Inspect Node State**

### **List All Nodes**
```bash
kubectl get nodes
```

### **Describe a Node**
```bash
kubectl describe node <node-name>
```
Provides detailed information, including:
- Allocated resources (CPU, memory).
- Running Pods.
- Labels and annotations.

### **Check Node Taints**
```bash
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

### **Node Status Summary**
```bash
kubectl top nodes
```
Shows resource usage (CPU, memory) for all nodes.

---

## **3. Analyze Pods**

### **List Pods with Wide Output**
```bash
kubectl get pods -o wide
```
Includes additional details such as node hosting the Pod and IP address.

### **Describe a Pod**
```bash
kubectl describe pod <pod-name>
```
Displays:
- Events (e.g., scheduling, failures).
- Container status (running, terminated, or failed).
- Restart counts and reasons.

### **Stream Pod Logs**
```bash
kubectl logs -f <pod-name>
```

### **Access a Running Pod**
```bash
kubectl exec -it <pod-name> -- /bin/bash
```
Open an interactive shell to debug inside the container.

---

## **4. Modify Resources**

### **Edit a Resource**
```bash
kubectl edit deployment <deployment-name>
```
Opens the resource definition in your default text editor (e.g., `vim`, `nano`).

- Useful for:
  - Updating image versions.
  - Modifying environment variables.

### **Patch a Resource**
```bash
kubectl patch deployment <deployment-name> --patch '{"spec": {"replicas": 5}}'
```
Directly modifies specific fields.

---

## **5. Explore Events and Logs**

### **List Cluster Events**
```bash
kubectl get events
```
Provides real-time insight into:
- Resource creation or updates.
- Errors or warnings (e.g., scheduling failures).

### **Follow Cluster Events**
```bash
kubectl get events --watch
```

---

## **6. Check Resource Utilization**

### **Pod Resource Usage**
```bash
kubectl top pod
```

### **Pod Usage in a Specific Namespace**
```bash
kubectl top pod -n <namespace>
```

---

## **7. Advanced Resource Exploration**

### **Dry-Run Mode**
- Simulates a command without making changes:
```bash
kubectl create deployment test-deploy --image=nginx --dry-run=client -o yaml
```

### **Resource Details with Custom Columns**
- Customize resource output:
```bash
kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName,STATUS:.status.phase
```

---

## **8. Node and Pod Scheduling**

### **Check Node Labels**
```bash
kubectl get nodes --show-labels
```

### **Add a Taint to a Node**
```bash
kubectl taint nodes <node-name> key=value:NoSchedule
```

### **Remove a Taint**
```bash
kubectl taint nodes <node-name> key=value:NoSchedule-
```

---

## **9. Cleanup Resources**

### **Delete Specific Resource**
```bash
kubectl delete pod <pod-name>
```

### **Delete All Pods in a Namespace**
```bash
kubectl delete pods --all
```

---

## **10. Additional Commands**

### **Port-Forward to Access a Pod**
```bash
kubectl port-forward pod/<pod-name> 8080:80
```
Forwards traffic from `localhost:8080` to the Pod’s port `80`.

### **Restart a Deployment**
```bash
kubectl rollout restart deployment <deployment-name>
```

### **View Rollout History**
```bash
kubectl rollout history deployment <deployment-name>
```
