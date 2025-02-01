
# Kubernetes Secrets Hands-On Lab

This lab will guide you through creating and using a **Secrets** in a Kubernetes cluster. By the end of this lab, you will understand how to store sensitive data in Secrets and inject it into Pods as environment variables, command-line arguments, or mounted files

---

## **Prerequisites**
- A running Kubernetes cluster (e.g., Minikube, Kind, or a cloud-based cluster).
- `kubectl` installed and configured.

---

## **Lab steps:**
### Create a secret
1. Create a Secret from Literal Values:
Use the `kubectl create secret` command to create a Secret from literal values:
	```bash
	kubectl create secret generic my-secret --from-literal=username=admin --from-literal=password=secret
2. Verify the Secret:
Check that the Secret was created successfully:
	```bash
	kubectl get secrets
	kubectl describe secret my-secret
3. View the Secret Data:
Secrets are stored in base64-encoded format. To view the decoded data:
	```bash
	kubectl get secret my-secret -o jsonpath="{.data.username}" | base64 --decode
	kubectl get secret my-secret -o jsonpath="{.data.password}" | base64 --decode
---
### Use Secret as Environment Variables
4. Create a Pod that Uses the Secret as Environment Variables:
Save the following YAML as pod-env.yaml:
	```yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  name: my-pod-env
	spec:
	  containers:
	  - name: my-container
	    image: nginx:stable-alpine
	    env:
	    - name: SECRET_USERNAME
	      valueFrom:
	        secretKeyRef:
	          name: my-secret
	          key: username
	    - name: SECRET_PASSWORD
	      valueFrom:
	        secretKeyRef:
	          name: my-secret
	          key: password
5. Apply the Pod:
Deploy the Pod to the cluster: `kubectl apply -f pod-env.yaml`
6. Verify the Environment Variables:
Check that the environment variables were injected correctly:  
`kubectl exec my-pod-env -- printenv SECRET_USERNAME SECRET_PASSWORD`
---
### Use Secret as Volumes
7. Save the following YAML as pod-cmd.yaml:
	```yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  name: my-pod-volume
	spec:
	  containers:
	  - name: my-container
	    image: nginx:stable-alpine
	    volumeMounts:
	    - name: secret-volume
	      mountPath: /etc/secret
	  volumes:
	  - name: secret-volume
	    secret:
	      secretName: my-secret
8. Apply the Pod: `kubectl apply -f pod-volume.yaml`
9. Check that the Secret files were mounted correctly:
	```bash
	kubectl exec my-pod-volume -- ls /etc/secret
	kubectl exec my-pod-volume -- cat /etc/secret/username
	kubectl exec my-pod-volume -- cat /etc/secret/password 
---
### Clean Up
10. Delete the Pods: `kubectl delete pod my-pod-env my-pod-volume`
11. Delete the secret: `kubectl delete secret my-secret`
---