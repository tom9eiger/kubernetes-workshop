# **Serve ConfigMap, Secret, and Host Information via NGINX**

This guide explains how to use Kubernetes ConfigMaps, Secrets, and Pod-specific information (hostname) to dynamically display data served by an NGINX Deployment with load balancing.

---

## **1. Create a Secret**

Create a Secret containing sensitive data:
```bash
kubectl create secret generic my-secret --from-literal=SECRET_VALUE=supersecret
```

Verify the Secret:
```bash
kubectl get secrets
kubectl describe secret my-secret
```

---

## **2. Create a ConfigMap**

Create a ConfigMap with an `index.html` file containing placeholders for the secret and hostname:
```bash
kubectl create configmap my-configmap --from-literal=index.html="<html><body><h1>My Secret is: \$SECRET_VALUE</h1><p>Served by: \$HOSTNAME</p></body></html>"
```

Verify the ConfigMap:
```bash
kubectl get configmaps
kubectl describe configmap my-configmap
```

---

## **3. Create a Deployment**

Define the following Deployment YAML file (`nginx-deployment.yaml`):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        image: nginx
        env:
        - name: SECRET_VALUE
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: SECRET_VALUE
        - name: HOSTNAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        volumeMounts:
        - name: config-volume
          mountPath: /usr/share/nginx/html/index.html
          subPath: index.html
      volumes:
      - name: config-volume
        configMap:
          name: my-configmap
```

- **Explanation of `HOSTNAME`:**  
  The `fieldRef` environment variable dynamically fetches the name of the Pod, which represents the hostname.

Apply the Deployment:
```bash
kubectl apply -f nginx-deployment.yaml
```

Verify the Pods:
```bash
kubectl get pods
```

---

## **4. Expose the Deployment**

Expose the Deployment as a NodePort Service:
```bash
kubectl expose deployment nginx-deployment --type=NodePort --port=80
```

Retrieve the service details:
```bash
kubectl get services
```

Retrieve the service URL using Minikube:
```bash
minikube service nginx-deployment --url
```

---

## **5. Access the Application**

Use `curl` or a browser to access the service and verify the output:
```bash
curl <SERVICE_URL>
```

The output will look similar to:
```html
<html>
  <body>
    <h1>My Secret is: supersecret</h1>
    <p>Served by: nginx-deployment-654c9f7c88-abc12</p>
  </body>
</html>
```

---

## **6. Scaling the Deployment**

Scale the Deployment to demonstrate load balancing:
```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Test multiple requests to observe the load balancing:
```bash
curl <SERVICE_URL>
curl <SERVICE_URL>
curl <SERVICE_URL>
```

The `Served by` field will show different Pod names for each request, demonstrating Kubernetes’ built-in load balancing.

---

## **7. Cleanup**

To remove all resources:
```bash
kubectl delete deployment nginx-deployment
kubectl delete service nginx-deployment
kubectl delete configmap my-configmap
kubectl delete secret my-secret
```

---

## **How It Works**

1. **ConfigMap**: Provides an HTML template (`index.html`) with placeholders for sensitive data (`$SECRET_VALUE`) and the Pod hostname (`$HOSTNAME`).
2. **Secret**: Injects sensitive information into the Pods as an environment variable.
3. **Dynamic Hostname**: Each Pod injects its name into the HTML, showcasing which Pod served the request.
4. **Load Balancing**: Kubernetes' Service distributes traffic across all Pods in the Deployment.
