# **Kubernetes Storage Workshop: RWO and RWX with PV and PVC**

This workshop demonstrates the use of **PersistentVolumes (PVs)** and **PersistentVolumeClaims (PVCs)** in Kubernetes with **ReadWriteOnce (RWO)** and **ReadWriteMany (RWX)** access modes. Minikube's default storage provisioner is used for dynamic volume provisioning.

---

## **1. Prerequisites**

- **Minikube** is installed and running:
  ```bash
  minikube start 
  ```
- Enable the default storage provisioner:
  ```bash
  minikube addons enable storage-provisioner
  ```

---

## **2. Example: ReadWriteOnce (RWO)**

### **Step 1: Create PersistentVolume and PVC**

**pv-rwo.yaml**:
```yaml
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
```

Apply the manifest:
```bash
kubectl apply -f pv-rwo.yaml
```

Verify that the resources were created:
```bash
kubectl get pv
kubectl get pvc
```

---

### **Step 2: Create Pod1 Using the PVC**

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

Verify the Pod is running:
```bash
kubectl get pods
```

---

### **Step 3: Attempt to Create Pod2 Using the Same PVC**

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

---

### **Step 4: Test Storage**

1. Access **Pod1**:
   ```bash
   kubectl exec -it pod1-rwo -- sh
   ```
2. Write data to the volume:
   ```sh
   echo "RWO Test Data" > /data/testfile.txt
   ```
3. Exit the Pod:
   ```bash
   exit
   ```
4. Access **Pod1**:
   ```bash
   kubectl exec -it pod2-rwo -- sh
   ```
5. Write data to the volume:
   ```sh
   ls /data/testfile.txt
   ```
6. Exit the Pod:
   ```bash
   exit
   ```
---

## **3. Example: ReadWriteMany (RWX)**

### **Step 1: Create PersistentVolume and PVC**

**pv-rwx.yaml**:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-rwx
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/rwx
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-rwx
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  volumeName: pv-rwx
```

Apply the manifest:
```bash
kubectl apply -f pv-rwx.yaml
```

Verify that the resources were created:
```bash
kubectl get pv
kubectl get pvc
```

---

### **Step 2: Create Two Pods Using the Same PVC**

**pod1-rwx.yaml**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1-rwx
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: "/data"
      name: shared-storage
  volumes:
  - name: shared-storage
    persistentVolumeClaim:
      claimName: pvc-rwx
```

**pod2-rwx.yaml**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod2-rwx
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: "/data"
      name: shared-storage
  volumes:
  - name: shared-storage
    persistentVolumeClaim:
      claimName: pvc-rwx
```

Apply both manifests:
```bash
kubectl apply -f pod1-rwx.yaml
kubectl apply -f pod2-rwx.yaml
```

Verify that both Pods are running:
```bash
kubectl get pods
```

---

### **Step 3: Test Shared Storage**

1. Access **Pod1**:
   ```bash
   kubectl exec -it pod1-rwx -- sh
   ```
2. Write data to the shared volume:
   ```bash
   echo "RWX Test Data" > /shared-data/testfile.txt
   ```
3. Exit Pod1:
   ```bash
   exit
   ```
4. Access **Pod2**:
   ```bash
   kubectl exec -it pod2-rwx -- sh
   ```
5. Verify the data:
   ```bash
   cat /shared-data/testfile.txt
   ```
   Expected output:
   ```plaintext
   RWX Test Data
   ```

---

## **4. Cleanup**

To remove all resources:
```bash
kubectl delete pod pod1-rwo pod2-rwo pod1-rwx pod2-rwx
kubectl delete pvc pvc-rwo pvc-rwx
kubectl delete pv pv-rwo pv-rwx
```

---

## **5. Key Takeaways**

- **RWO (ReadWriteOnce):** Allows one Pod to mount the volume at a time.
- **RWX (ReadWriteMany):** Allows multiple Pods to share the same volume.
- Minikube’s **hostPath** simplifies testing PVs and PVCs locally.
