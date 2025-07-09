# 🧪 Hands-On Lab: Exposing a Kubernetes Deployment Using a NodePort Service

## 🎯 Objective
- Deploy a simple web application using a Kubernetes Deployment
- Expose the deployment using a NodePort Service
- Access the application externally via the Node’s IP and assigned port

---

## 🔧 Prerequisites
- A running Kubernetes cluster (Minikube, KIND, or a cloud-based cluster)
- `kubectl` installed and configured

---

## 📌 Step 1: Create a Deployment

Create a file named `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

Apply the deployment:

```bash
kubectl apply -f deployment.yaml
```

---

## 📌 Step 2: Expose the Deployment via NodePort

Create a file named `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # Choose a port between 30000–32767
```

Apply the service:

```bash
kubectl apply -f service.yaml
```

---

## 📌 Step 3: Access the Web Application

### For Minikube users:
```bash
minikube ip
```

Navigate to:

```
http://<minikube-ip>:30080
```

You should see the **NGINX welcome page**.

---

## 🧹 Cleanup

```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

---

## ❓ Lab Review Questions

1. What does `type: NodePort` do in the service definition?
2. What is the purpose of `targetPort` and `nodePort`?
3. What happens if you omit the `nodePort` value?
4. Can a NodePort service expose a Pod running on a specific node only?
5. How does this differ from `LoadBalancer` or `ClusterIP` service types?