
# Kubernetes DaemonSet Hands-On Lab

This lab will guide you through creating and managing a **DaemonSet** in a Kubernetes cluster. By the end of this lab, you will understand how DaemonSets work and how to schedule them on specific nodes using `nodeSelector` and `tolerations`.

---

## **Prerequisites**
- A running Kubernetes cluster with at least 2 nodes (e.g., Minikube with multiple nodes, Kind, or a cloud-based cluster).
- `kubectl` installed and configured.

---

## **Lab steps:**

1. Create minikube cluster with 2 nodes `minikube start --nodes 2 -p multinode-demo`
2. Ensure the 2 nodes are initialized `kubectl get nodes`
3. Add a label to a specific node where the DaemonSet should run:
   ```bash
   kubectl label node <node-name> environment=production
4. Verify the label `kubectl get nodes --show-labels | grep environment`
5. Save the following YAML as daemonset.yaml
	```yaml
	apiVersion: apps/v1
	kind: DaemonSet
	metadata:
	  name: my-daemonset
	spec:
	  selector:
	    matchLabels:
	      app: logging-agent
	  template:
	    metadata:
	      labels:
	        app: logging-agent
	    spec:
	      containers:
	      - name: fluentd
	        image: fluentd:v1.18.0-debian-1.0
	        resources:
	          limits:
	            memory: "200Mi"
	            cpu: "500m"
	        volumeMounts:
	        - name: varlog
	          mountPath: /var/log
	      volumes:
	      - name: varlog
	        hostPath:
	          path: /var/log
	      nodeSelector:
	        environment: production  # Schedule only on nodes with this label
6. Create daemonset with the `nodeSelector`
Apply the DaemonSet: `kubectl apply -f daemonset.yaml`
8. Check the status of the DaemonSet: `kubectl get daemonset my-daemonset`
9. List the Pods created by the DaemonSet: `kubectl get pods -l app=logging-agent -o wide`  
> Observe that the Pods are only running on the node(s) with the environment=production label
### use taints and tolerations
10. Taint a node to prevent Pods from being scheduled on it unless they have a matching toleration: `kubectl taint nodes <node-name> key=value:NoSchedule`
> example: `kubectl taint nodes node-2 app=logging:NoSchedule`
11. Update the DaemonSet to include a toleration for the taint:
	```yaml
	  template:
	    spec:
	      tolerations:
	      - key: "app"
	        operator: "Equal"
	        value: "logging"
	        effect: "NoSchedule"
12. Apply the updated DaemonSet: `kubectl apply -f daemonset.yaml`
13. Verify Tolerations:
Check the status of the DaemonSet: `kubectl get daemonset my-daemonset`
List the Pods created by the DaemonSet: `kubectl get pods -l app=logging-agent -o wide`
> Observe that the Pods are now running on both the labeled node (environment=production) and the tainted node (app=logging:NoSchedule)
14. Change the toleration.key to db instead of app, redploy the daemonset `kubectl apply -f daemonset.yaml` and then list the pods `kubectl get pods -l app=logging-agent -o wide`
> Observe the only daemonset pod running
### Clean up
15. Delete the DaemonSet: `kubectl delete daemonset my-daemonset`
16. Remove the node label: `kubectl label node <node-name> environment-`
17. Remove node taint: `kubectl taint nodes <node-name> key:NoSchedule-`