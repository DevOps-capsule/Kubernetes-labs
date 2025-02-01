
# Kubernetes ConfigMap Hands-On Lab

This lab will guide you through creating and using a **ConfigMap** in a Kubernetes cluster. By the end of this lab, you will understand how to store configuration data in ConfigMaps and inject it into Pods as environment variables, command-line arguments, or mounted files.

---

## **Prerequisites**
- A running Kubernetes cluster (e.g., Minikube, Kind, or a cloud-based cluster).
- `kubectl` installed and configured.

---

## **Lab steps:**

1. Save the following YAML as `configmap.yaml`:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: my-configmap
   data:
     key1: value1
     key2: value2
     app-config.properties: |
       property1=value1
       property2=value2
2. Apply the ConfigMap: `kubectl apply -f configmap.yaml`
3. Verify the ConfigMap:
	```bash
		kubectl get configmap my-configmap
		kubectl describe configmap my-configmap
### Use ConfigMap as Environment Variables
4. Save the following YAML as pod-env.yaml:
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
	    - name: KEY1
	      valueFrom:
	        configMapKeyRef:
	          name: my-configmap
	          key: key1
5. Apply the Pod: `kubectl apply -f pod-env.yaml`
6. Verify the environment variable: `kubectl exec my-pod-env -- printenv KEY1`
### Use ConfigMap as Command-Line Arguments
7. Save the following YAML as pod-cmd.yaml:
	```yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  name: my-pod-cmd
	spec:
	  restartPolicy: Never
	  containers:
	  - name: my-container
	    image: busybox
	    command: ["echo", "$(KEY1)"]
	    env:
	    - name: KEY1
	      valueFrom:
	        configMapKeyRef:
	          name: my-configmap
	          key: key1
8. Apply the Pod: `kubectl apply -f pod-cmd.yaml`
9. Check the logs to verify the command-line argument: `kubectl logs my-pod-cmd`  
Expected output: `value1`
### Use ConfigMap as Volumes
10. Save the following YAML as pod-volume.yaml:
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
	    - name: config-volume
	      mountPath: /etc/config
	  volumes:
	  - name: config-volume
	    configMap:
	      name: my-configmap
11. Apply the Pod: `kubectl apply -f pod-volume.yaml`
12. Verify the mounted configuration file: `kubectl exec my-pod-volume -- cat /etc/config/app-config.properties`
### Clean Up
13. Delete the Pods: `kubectl delete pod my-pod-env my-pod-cmd my-pod-volume`
14. Delete the ConfigMap: `kubectl delete configmap my-configmap`