
# Kubernetes Statefulset Hands-On Lab

  

In this lab, you’ll learn how to create, manage, and use Kubernetes Namespaces to organize and isolate resources in a cluster. You’ll also explore features like Resource Quotas, Limit Ranges, and Role-Based Access Control (RBAC) within namespaces

---
## **Objective**

In this lab, you will:
1.  Create and manage namespaces
2.  Deploy applications in different namespaces
3.  Apply resource quotas and limit ranges
4.  Use RBAC to restrict access to namespaces
---
## **Prerequisites**

- A running Kubernetes cluster (e.g., Minikube with multiple nodes, Kind, or a cloud-based cluster).

-  `kubectl` installed and configured.

---
## **Lab steps:**
### ## **Create Namespaces**
1. Create Namespaces Using kubectl
Create two namespaces, dev and prod, using the kubectl command:  
	```bash
	kubectl create namespace dev
	kubectl create namespace prod
2. Verify Namespaces: List all namespaces to verify the creation:  
`kubectl get namespaces` expected output:
	```bash
	NAME              STATUS   AGE
	default           Active   10d
	kube-system       Active   10d
	kube-public       Active   10d
	kube-node-lease   Active   10d
	dev               Active   10s
	prod              Active   10s
### Deploy applications in Namespaces
3. Deploy an Application in the dev Namespace  
Create a file named `app-dev.yam`l with the following content:
	```yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: nginx-dev
	  namespace: dev
	spec:
	  replicas: 2
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
Apply the manifest: `kubectl apply -f app-dev.yaml`  

4. Deploy an application in the prod Namespace  
Create a file named `app-prod.yaml` with the following content:
	```yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: nginx-prod
	  namespace: prod
	spec:
	  replicas: 2
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
Apply the manifest: `kubectl apply -f app-prod.yaml`  
### **Verify Deployments**
5. Check the Pods in each namespace:
	```bash
	kubectl get pods -n dev
	kubectl get pods -n prod
### Apply resource quotas
6. Create a Resource Quota for the dev Namespace
Create a file named `quota-dev.yaml` with the following content:
	```yaml
	apiVersion: v1
	kind: ResourceQuota
	metadata:
	  name: dev-quota
	  namespace: dev
	spec:
	  hard:
	    requests.cpu: "1"
	    requests.memory: "1Gi"
	    limits.cpu: "2"
	    limits.memory: "2Gi"  
Apply the quota: `kubectl apply -f quota-dev.yaml`  

7. Verify the Resource Quota
Check the resource quota in the dev namespace `kubectl get resourcequota -n dev`
Expected output:
	```bash
	NAME        AGE   REQUEST                                                                   LIMIT
	dev-quota   10s   requests.cpu: 0/1, requests.memory: 0/1Gi, limits.cpu: 0/2, limits.memory: 0/2Gi
### Apply Limit Ranges
8. Create a Limit Range for the prod Namespace
Create a file named `limit-range-prod.yaml` with the following content:
	```yaml
	apiVersion: v1
	kind: LimitRange
	metadata:
	  name: prod-limit-range
	  namespace: prod
	spec:
	  limits:
	  - default:
	      cpu: "500m"
	      memory: "512Mi"
	    defaultRequest:
	      cpu: "250m"
	      memory: "256Mi"
	    type: Container
Apply the limit range: `kubectl apply -f limit-range-prod.yaml`  

9. Verify the Limit Range  
Check the limit range in the prod namespace: `kubectl get limitrange -n prod`
### Use RBAC to Restrict Access
10. Create a service account in dev namespace:  `kubectl create serviceaccount my-service-account -n dev`  
Verify service account creation  
`kubectl get serviceaccount my-service-account -n default -o yaml`
11. Create a Role in the dev Namespace
Create a file named `role-dev.yaml` with the following content:
	```yaml
	apiVersion: rbac.authorization.k8s.io/v1
	kind: Role
	metadata:
	  name: pod-reader
	  namespace: dev
	rules:
	- apiGroups: [""]
	  resources: ["pods"]
	  verbs: ["get", "list", "watch"]  
	  
Apply the Role: `kubectl apply -f role-dev.yaml`  

12. Bind the Role to the service account `my-service-account`  
Create a file named role-binding-dev.yaml with the following content:
	```yaml
	apiVersion: rbac.authorization.k8s.io/v1
	kind: RoleBinding
	metadata:
	  name: read-pods
	  namespace: dev
	subjects:
	- kind: ServiceAccount
	  name: my-service-account
	  apiGroup: rbac.authorization.k8s.io
	roleRef:
	  kind: Role
	  name: pod-reader
	  apiGroup: rbac.authorization.k8s.io
Apply the RoleBinding: `kubectl apply -f role-binding-dev.yaml`  

13. Permission Verification Command: 
`kubectl auth can-i list pods --as=system:serviceaccount:default:my-service-account`  
Expected output is `yes`
### Clean up
14. Delete Namespaces
Delete the dev and prod namespaces:
	```bash
	kubectl delete namespace dev
	kubectl delete namespace prod`
15. Verify Deletion
List all namespaces to confirm deletion: `kubectl get namespaces`