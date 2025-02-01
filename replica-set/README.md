# Kubernetes ReplicaSet Hands-On Lab

This lab will guide you through creating and managing a **ReplicaSet** in a Kubernetes cluster. By the end of this lab, you will understand how ReplicaSets work and how to perform common operations like scaling and self-healing.

---

## **Prerequisites**
- A running Kubernetes cluster (e.g., Minikube, Kind, or a cloud-based cluster).
- `kubectl` installed and configured.

---

## **Steps**

1. Save the following YAML as `replicaset.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
     name: my-replicaset
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: nginx-app
     template:
       metadata:
         labels:
           app: nginx-app
       spec:
         containers:
         - name: nginx-container
           image: nginx:stable-alpine

2. Apply the YAML to create the ReplicaSet:

    ```bash
    kubectl apply -f replicaset.yaml

3. Verify the ReplicaSet and Pods:
    ```bash
    kubectl get replicaset
    kubectl get pods -l app=nginx-app

4. Scale the replicaset and then check the new pods
    ```bash
    kubectl scale replicaset my-replicaset --replicas=5
    kubectl get pods -l app=nginx-app

5. Test self healing  
open 2 terminals
    ```bash
    kubectl get pods -wl app=nginx-app # always watching the pods in terminal 1
    kubectl delete pod <pod-name>   # delete a pod in terminal 2 and go back to terminal 1 to observe

6. Create a service to access your application  
Save the following YAML as `service.yaml`:
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      selector:
        app: nginx-app
      ports:
      - protocol: TCP
        port: 80
        targetPort: 80
      type: ClusterIP

7. Deploy the service `kubectl apply -f service.yaml`
8. Verify the service and look for ports `kubectl get service my-service`  
9. get minikube ip, look for internal IP `kubectl get node -o wide`
10. Test the application using `<minikube-ip>:<svc-port>` through curl or the browser
11. Clean up:
    ```bash
    kubectl delete replicaset my-replicaset     # delete replicaset
    kubectl delete service my-service           # delete the service