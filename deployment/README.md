# **Hands-On Lab: Kubernetes Deployment**

In this lab, you will learn how to create, manage, and scale a Kubernetes Deployment. Deployments are the recommended way to manage stateless applications in Kubernetes, as they provide features like rolling updates, rollbacks, and self-healing.

---

## **Lab Objectives**
1. Create a Deployment.
2. Scale the Deployment.
3. Perform a rolling update.
4. Rollback the Deployment.
5. Delete the Deployment.

---

## **Prerequisites**
- A running Kubernetes cluster (e.g., Minikube, Kind, or a cloud-based cluster).
- `kubectl` installed and configured to interact with your cluster.

---

## **Lab steps**

1. Create a Deployment YAML File
Create a file named `nginx-deployment.yaml` with the following content:

	```yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: nginx-deployment
	  labels:
	    app: nginx
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
	        image: nginx:stable-alpine
	        ports:
	        - containerPort: 80

2. Create the Deployment**
Run the following command to create the Deployment:
`kubectl apply -f nginx-deployment.yaml`
3. Check the status of the Deployment:** `kubectl get deployments`
4. Check the replicasets and pods created by the deployment**
	```bash
		kubectl get replicasets
		kubectl get pods
5. Increase the number of replicas to 5: `kubectl scale deployment nginx-deployment --replicas=5`
6.  Perform a Rolling Update: `kubectl set image deployment/nginx-deployment nginx=nginx:alpine3.20`
7.  Monitor the Rolling Update: `kubectl rollout status deployment/nginx-deployment`
8. Expected output
	```bash
	Waiting for rollout to finish: 2 out of 5 new replicas have been updated...
	Waiting for rollout to finish: 3 out of 5 new replicas have been updated...
	Waiting for rollout to finish: 4 out of 5 new replicas have been updated...
	Waiting for rollout to finish: 5 out of 5 new replicas have been updated...
	deployment "nginx-deployment" successfully rolled out
9. Verify the Update: `kubectl describe pod <any-pod-name>` and look for container image in the events to see the new image name
10. Rollback the Deployment
Check rollout history: `kubectl rollout history deployment/nginx-deployment`
Rollback to the Previous Version: `kubectl rollout undo deployment/nginx-deployment`
11. Verify the Rollback: `kubectl rollout status deployment/nginx-deployment`
12. Check the Pods to confirm the rollback: `kubectl get pods`
13. Clean up:
Delete the deployment `kubectl delete deployment nginx-deployment`