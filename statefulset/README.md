
# Kubernetes Statefulset Hands-On Lab

  

This lab will guide you through creating and managing a **Statefulset** resource in a Kubernetes cluster. By the end of this lab, you will understand how Statefulset work.

---
## **Objective**

In this lab, you will:
1.  Create a  **StatefulSet**  for a stateful application (e.g., a database).
2.  Understand how StatefulSets provide  **stable network identities**  and  **persistent storage**.
3.  Explore  **ordered deployment**,  **scaling**, and  **rolling updates**.
4.  Clean up resources after the lab
---
## **Prerequisites**

- A running Kubernetes cluster (e.g., Minikube with multiple nodes, Kind, or a cloud-based cluster).

-  `kubectl` installed and configured.

---
## **Lab steps:**
### Creating the Statefulset and the Headless service
1. Create a Headless Service:
A headless service is required to provide stable network identities for the pods in the StatefulSet
	```yaml
	apiVersion: v1
	kind: Service
	metadata:
	  name: my-service
	spec:
	  clusterIP: None  # Headless service
	  selector:
	    app: my-app
	  ports:
	  - port: 80
	    targetPort: 80
2. Apply `kubectl apply -f headless-service.yaml`
Verify the service `kubectl get svc my-service`
3. Create a StatefulSet with persistent storage and ordered deployment
	```yaml
	apiVersion: apps/v1
	kind: StatefulSet
	metadata:
	  name: my-statefulset
	spec:
	  serviceName: "my-service"  # Headless service name
	  replicas: 3                # Number of pods
	  selector:
	    matchLabels:
	      app: my-app
	  template:
	    metadata:
	      labels:
	        app: my-app
	    spec:
	      containers:
	      - name: my-container
	        image: nginx:stable-alpine  # Use a simple web server for demonstration
	        ports:
	        - containerPort: 80
	        volumeMounts:
	        - name: my-storage
	          mountPath: /usr/share/nginx/html  # Mount path for persistent storage
	  volumeClaimTemplates:      # Dynamic provisioning of persistent storage
	  - metadata:
	      name: my-storage
	    spec:
	      accessModes: [ "ReadWriteOnce" ]
	      resources:
	        requests:
	          storage: 1Gi       # 1GB of storage per pod
4. Apply `kubectl apply -f headless-service.yaml`  
Verify the statefulset `kubectl get statefulset my-statefulset`  
Verify the Pods `kubectl get pods -l app=my-app`
---
### Explore Stable Network Identities
5. Check the DNS names of the pods: `kubectl run -it --rm --image=busybox:latest --restart=Never test-pod -- sh`  
Inside the test-pod, run: `nslookup my-service`  
You should see DNS entries for each pod (e.g., `my-statefulset-0.my-service`, `my-statefulset-1.my-service`)
---
### Explore Persistent Storage
6. Verify the PersistentVolumeClaims (PVCs): `kubectl get pvc`  
Each pod in the StatefulSet gets its own persistent volume  
> **You should see PVCs like my-storage-my-statefulset-0, my-storage-my-statefulset-1, etc.**
7. Write data to the persistent volume of a pod:  
`kubectl exec my-statefulset-0 -- sh -c "echo 'Hello from pod-0' > /usr/share/nginx/html/index.html"`
8. Verify the data: `kubectl exec my-statefulset-0 -- cat /usr/share/nginx/html/index.html`
---
### Scale the StatefulSet
9. StatefulSets support ordered scaling
Scale up the StatefulSet to 5 replicas: `kubectl scale statefulset my-statefulset --replicas=5`  
Observe the ordered creation of new pods: `kubectl get pods -l app=my-app -w`  
Scale down the StatefulSet to 2 replicas: `kubectl scale statefulset my-statefulset --replicas=2`  
Observe the ordered deletion of pods (starting from the highest ordinal) `kubectl get pods -l app=my-app -w`
---
### Perform Rolling Updates
10. Update the container image in the StatefulSet: to `nginx:latest`  
StatefulSets support controlled rolling updates  
Observe the rolling update: `kubectl get pods -l app=my-app -w`
> **Pods are updated in reverse ordinal order (`my-statefulset-2`, `my-statefulset-1`, `my-statefulset-0`)**
### Clean up
11. Delete the resources created during the lab  
Delete the StatefulSet: `kubectl delete statefulset my-statefulset`  
Delete the headless service: `kubectl delete svc my-service`  
Delete the PVCs (if not automatically deleted): `kubectl delete pvc -l app=my-app`