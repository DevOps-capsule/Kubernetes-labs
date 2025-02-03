# Kubernetes Ingress Hands-On Lab

  

This lab will guide you through creating and managing a **Ingress** controller and resource in a Kubernetes cluster. By the end of this lab, you will understand how Ingress work and how to expose multiple services/apps through a single LoadBalancer.

---

## **Prerequisites**

- A running Kubernetes cluster (e.g., Minikube with multiple nodes, Kind, or a cloud-based cluster).

-  `kubectl` installed and configured.

---
## **Lab steps:**
### Configuring the controller
1. Install the NGINX Ingress controller
You’ll install the NGINX controller from a YAML file hosted in the Kubernetes GitHub repo. It installs a bunch of Kubernetes constructs, including a Namespace, ServiceAccounts, ConfigMap, Roles, RoleBindings, and more.  
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/cloud/deploy.yaml`  
**In case of Minikube run `minikube addons enable ingress`**  

2. Run the following command to check the ingress-nginx Namespace and ensure the
controller Pod is running. It may take a few seconds to enter the running phase  
`kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx`
	> Don’t worry about the Completed Pods. These were short-lived Pods that initialized the environment. Once the controller Pod is running, you have an NGINX Ingress controller and are ready to create some Ingress objects  
    
3. If you’re following along, you’ll have at least one Ingress class called nginx. This was created when you installed the NGINX controller: `kubectl get ingressclass`

### Creating the two backend pods, services and ingress
4. Create the first backend file `backend-1.yaml` with the following code:
	```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: backend1
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: backend1
    template:
        metadata:
        labels:
            app: backend1
        spec:
        containers:
        - name: catgif
            image: alaaamin/catnip
            ports:
            - containerPort: 5000
    ---
    apiVersion: v1
    kind: Service
    metadata:
    name: catgif-clusterip
    spec:
    selector:
        app: backend1
    ports:
    - protocol: TCP
        port: 6000
        targetPort: 5000
5. Apply the deployment `kubectl apply -f backend-1.yaml`
6. Create the second backend file `backend-2.yaml` with the following code:
	```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: backend2
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: backend2
    template:
        metadata:
        labels:
            app: backend2
        spec:
        containers:
        - name: nginx
            image: nginx:stable-alpine
            ports:
            - containerPort: 80
    ---
    apiVersion: v1
    kind: Service
    metadata:
    name: nginx-clusterip
    spec:
    selector:
        app: backend2
    ports:
    - protocol: TCP
        port: 8080
        targetPort: 80
7. Apply the deployment `kubectl apply -f backend-2.yaml`
8. Create the Ingress file `ingress.yaml` with the following code:
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: example-ingress
    annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
    ingressClassName: nginx
    rules:
    - host: catgif.mcu.com  # <== Host rule for catgif app  # Traffic arriving via this hostname
        http:
        paths:
        - path: /
            pathType: Prefix
            backend:
            service:            # Refrencing the existing service for the catgif app
                name: catgif-clusterip
                port:
                number: 6000    # Service exposed port
    - host: nginx.mcu.com   # <==== Host rule for nginx server app
        http:
        paths:
        - path: /
            pathType: Prefix
            backend:
            service:            # Refrencing the existing service for the nginx server
                name: nginx-clusterip
                port:
                number: 8080    # Service exposed port
    - host: mcu.com
        http:
        paths:
        - path: /catgif       # <==== Path rule for catgif app
            pathType: Prefix
            backend:
            service:
                name: catgif-clusterip
                port:
                number: 6000
        - path: /nginx       # <==== Path rule for nginx app
            pathType: Prefix
            backend:
            service:
                name: nginx-clusterip
                port:
                number: 8080

9. Apply the ingress object `kubectl apply -f ingress.yaml`

### Walkthrough and configuration validation
10. List all Ingress objects in the default Namespace ` kubectl get ing`  
The `CLASS` field shows which Ingress class is handling this set of rules.  
It may show as `<None>` if you only have a single Ingress controller and didn’t configure classes.  
The `HOSTS` field is a list of hostnames the Ingress will handle traffic for.  
The `ADDRESS` field is the load balancer endpoint

    **> On the topic of ports, Ingress only supports HTTP and HTTPS**  

11. Examine the ingress describe output `kubectl describe ing example-ingress`  
- The `Address` line is the IP or DNS name of the load balancer created by the Ingress. It might be localhost on local clusters
- Default backend is where the controller sends traffic arriving on a hostname or path it doesn’t have a route for. Not all Ingress controllers implement a default backend
- The rules define the mappings between hosts, paths, and backends. Remember that backends are usually ClusterIP Services that send traffic to Pods
- You can use annotations to define controller-specific features and integrations with your cloud back end
- This example tells the controller to rewrite all paths to look like they arrived on root “/”

### Configuring the DNS
12. **Configure DNS name resolution**  
In the real world, you’ll configure your internal DNS or internet DNS to point hostnames to the Ingress load balancer  
How you do this varies depending on your environment and who provides your internet DNS  
  
The easiest thing to do is edit the hosts file on your local computer and map the hostnames to the Ingress load balancer
- On Mac and Linux, this file is `/etc/hosts`, and you’ll need root permissions to edit it.  
On Windows, it’s `C:\Windows\System32\drivers\etc\hosts`, and you’ll need to open it as an administrator
- `sudo nano /etc/hosts`
- add the following lines
    ```text
    <ingress-ip-addr-in step 10> catgif.mcu.com
    <ingress-ip-addr-in step 10> nginx.mcu.com
    <ingress-ip-addr-in step 10> mcu.com
- Save and exit by pressing `ctrl + o` and then `ctrl + x`

13. Test the Ingress
Open a web browser and try the following URLs:
- shield.mcu.com
- hydra.mcu.com
- mcu.com/catgif
- mcu.com/nginx
- mcu.com

### Clean-up
14. `kubectl delete -f backend-1.yaml`
15. `kubectl delete -f backend-2.yaml`
16. `kubectl delete -f ingress.yaml`
17. Optional disable the INgress controller`minikube addons disable ingress`
18. Remove the DNS configuration lines made in step 12 from the file `/etc/hosts`