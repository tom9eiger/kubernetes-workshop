
---

# **Practical Tasks for CoreDNS in Kubernetes**

This guide provides common tasks and exercises to understand and work with CoreDNS, the default DNS provider in Kubernetes. It now includes an example of creating an ExternalName Service.

---

## **1. Objectives**

- Understand how CoreDNS works in Kubernetes.
- Test DNS resolution for Services and Pods.
- Configure and troubleshoot CoreDNS.
- Resolve external DNS using ExternalName Services.

---

## **2. Prerequisites**

- A running Kubernetes cluster (e.g., Minikube, K3s, or a cloud provider).
- `kubectl` is installed and configured.

---

## **3. Tasks**

### **Task 1: Verify CoreDNS is Running**

1. List CoreDNS Pods:
   ```bash
   kubectl get pods -n kube-system -l k8s-app=kube-dns
   ```

2. Check CoreDNS logs:
   ```bash
   kubectl logs -n kube-system -l k8s-app=kube-dns
   ```

3. Describe the CoreDNS Deployment:
   ```bash
   kubectl describe deployment -n kube-system coredns
   ```

---

### **Task 2: Test DNS Resolution**

1. Deploy a Service:
   Save the following YAML as `my-service.yaml`:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     selector:
       app: my-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
   ```

   Apply the YAML:
   ```bash
   kubectl apply -f my-service.yaml
   ```

2. Deploy a Pod:
   Save the following YAML as `test-pod.yaml`:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test-pod
   spec:
     containers:
     - name: test-container
       image: busybox
       command: ["sleep", "3600"]
   ```

   Apply the YAML:
   ```bash
   kubectl apply -f test-pod.yaml
   ```

3. Test DNS resolution from the Pod:
   ```bash
   kubectl exec -it test-pod -- nslookup my-service
   kubectl exec -it test-pod -- ping my-service
   ```

---

### **Task 3: Test Cross-Namespace DNS**

1. Create a new namespace and deploy a Service:
   ```bash
   kubectl create namespace test-ns
   kubectl apply -f my-service.yaml -n test-ns
   ```

2. Query the Service from the `default` namespace:
   ```bash
   kubectl exec -it test-pod -- nslookup my-service.test-ns.svc.cluster.local
   ```

---

### **Task 4: Configure CoreDNS for External Domains**

1. Edit the CoreDNS ConfigMap:
   ```bash
   kubectl edit configmap -n kube-system coredns
   ```

2. Add the following block to enable external domain forwarding:
   ```yaml
   forward . 8.8.8.8
   ```

3. Save the ConfigMap and restart CoreDNS Pods:
   ```bash
   kubectl rollout restart deployment -n kube-system coredns
   ```

4. Test external domain resolution:
   ```bash
   kubectl exec -it test-pod -- nslookup example.com
   ```

---

### **Task 5: Use an ExternalName Service**

1. Create an ExternalName Service to resolve `example.com`:
   Save the following YAML as `external-service.yaml`:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: external-service
   spec:
     type: ExternalName
     externalName: example.com
   ```

   Apply the YAML:
   ```bash
   kubectl apply -f external-service.yaml
   ```

2. Test the ExternalName Service:
   ```bash
   kubectl exec -it test-pod -- nslookup external-service
   kubectl exec -it test-pod -- curl external-service
   ```

3. Verify that the Service resolves to the external domain `example.com`.

---

### **Task 6: Debug DNS Issues**

1. Check the DNS Policy of a Pod:
   ```bash
   kubectl get pod test-pod -o yaml | grep dnsPolicy
   ```

2. Test DNS resolution using `dig` (if installed):
   ```bash
   kubectl exec -it test-pod -- dig my-service
   ```

3. Check CoreDNS metrics (if enabled):
   ```bash
   kubectl port-forward -n kube-system svc/coredns 9153:9153
   curl http://localhost:9153/metrics
   ```

---

## **4. Cleanup**

To remove all resources created during the tasks:
```bash
kubectl delete -f my-service.yaml
kubectl delete -f test-pod.yaml
kubectl delete -f external-service.yaml
kubectl delete namespace test-ns
```

---

## **5. Key Points**

- CoreDNS resolves internal Kubernetes Services using DNS names.
- ExternalName Services map Kubernetes Service names to external DNS domains.
- Cross-namespace resolution requires fully qualified domain names.
- External DNS resolution can be configured in the CoreDNS ConfigMap.

