
**Title: Hands-On Lab: Persistent Volumes, Persistent Volume Claims, and Storage Classes in Kubernetes**

**Objective:**  
In this lab, you'll learn how to manually and automatically provision persistent storage in Kubernetes using Persistent Volumes (PVs), Persistent Volume Claims (PVCs), and StorageClasses.

----------

## **1. Prerequisites**

-   Kubernetes cluster (Minikube or remote)
-   kubectl configured and working

----------

## **2. Manually Provisioning Storage Using PV and PVC**

### **Step 1: Create a Persistent Volume (PV)**

```yaml
# pv-manual.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

```bash
kubectl apply -f pv-manual.yaml
```

### **Step 2: Create a Persistent Volume Claim (PVC)**

```yaml
# pvc-manual.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: manual-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
kubectl apply -f pvc-manual.yaml
```

### **Step 3: Deploy a Pod Using the PVC**

```yaml
# pod-manual.yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - mountPath: "/data"
          name: manual-storage
  volumes:
    - name: manual-storage
      persistentVolumeClaim:
        claimName: manual-pvc
```

```bash
kubectl apply -f pod-manual.yaml
```

### **Step 4: Verify**

```bash
kubectl get pv
kubectl get pvc
kubectl describe pod manual-pod
```

----------

## **3. Automatically Provisioning Storage Using PVC and StorageClass**

### **Step 1: Enable Default StorageClass (Minikube example)**

```bash
minikube addons enable storage-provisioner	# If not already enabled
```

### **Step 2: Create a PVC with a StorageClass**

```yaml
# pvc-auto.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: auto-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
```

```bash
kubectl apply -f pvc-auto.yaml
```

### **Step 3: Deploy a Pod Using the Auto-Provisioned PVC**

```yaml
# pod-auto.yaml
apiVersion: v1
kind: Pod
metadata:
  name: auto-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - mountPath: "/data"
          name: auto-storage
  volumes:
    - name: auto-storage
      persistentVolumeClaim:
        claimName: auto-pvc
```

```bash
kubectl apply -f pod-auto.yaml
```

### **Step 4: Verify**

```bash
kubectl get pvc
kubectl get pv
kubectl describe pod auto-pod
```

----------

## **4. Cleanup**

```bash
kubectl delete -f pod-manual.yaml
kubectl delete -f pvc-manual.yaml
kubectl delete -f pv-manual.yaml
kubectl delete -f pod-auto.yaml
kubectl delete -f pvc-auto.yaml
```

----------

**Conclusion:**  
You now understand how to provision storage in Kubernetes both manually and automatically. Persistent Volumes and Persistent Volume Claims decouple storage configuration from pod definitions, and StorageClasses simplify dynamic provisioning in production environments.