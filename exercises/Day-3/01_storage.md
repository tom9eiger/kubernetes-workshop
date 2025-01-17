# **Workshop: Storage in Kubernetes with Minikube**

---

## **1. Prerequisites**

- Minikube installed and running:
  ```bash
  minikube start
  ```
- Enable the storage provisioner:
  ```bash
  minikube addons enable storage-provisioner
  ```

---

## **2. ReadWriteOnce (RWO) Example**

### **Step 1: Create PersistentVolume and PersistentVolumeClaim**

**pvc-rwo.yaml**:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-rwo
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data-rwo
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-rwo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  volumeName: pv-rwo
```

Apply the manifest:
```bash
kubectl apply -f pvc-rwo.yaml
```

Verify the resources:
```bash
kubectl get pv
kubectl get pvc
```

---

### **Step 2: Create Pod1 Using the RWO PVC**

**pod1-rwo.yaml**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1-rwo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-rwo
```

Apply the manifest:
```bash
kubectl apply -f pod1-rwo.yaml
```

Verify the Pod:
```bash
kubectl get pods
```

---

### **Step 3: Attempt to Create Pod2 Using the Same RWO PVC**

**pod2-rwo.yaml**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod2-rwo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-rwo
```

Apply the manifest:
```bash
kubectl apply -f pod2-rwo.yaml
```

Expected outcome:
- **Pod1** will continue running without issues.
- **Pod2** will fail to start, as the PVC is already in use by Pod1, demonstrating the restriction of the **RWO** mode.

Check the Pod statuses:
```bash
kubectl get pods
```

Check events for Pod2:
```bash
kubectl describe pod pod2-rwo
```

---

### **Step 4: Test Storage with Pod1**

1. Access Pod1:
   ```bash
   kubectl exec -it pod1-rwo -- sh
   ```
2. Write a file:
   ```sh
   echo "RWO Test Data" > /data/testfile.txt
   ```
3. Exit the Pod:
   ```bash
   exit
   ```

---

## **3. ReadWriteMany (RWX) Example**

Follow the steps outlined earlier for **RWX** with the additional **pod1-rwx** and **pod2-rwx** examples to demonstrate shared access to the same volume.

---

## **4. Cleanup**

Remove all resources:
```bash
kubectl delete pod pod1-rwo pod2-rwo pod1-rwx pod2-rwx
kubectl delete pvc pvc-rwo pvc-rwx
kubectl delete pv pv-rwo pv-rwx
```

---

## **Key Takeaways**

- **RWO (ReadWriteOnce):** A PVC can only be mounted by one Pod at a time.  
- **RWX (ReadWriteMany):** A PVC can be shared across multiple Pods and Nodes.  
- **Pod2 in RWO:** Fails to run if the PVC is already attached to another Pod, ensuring data integrity.