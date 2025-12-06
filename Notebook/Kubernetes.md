It is a container orchestration tool. It helps manage containerized applications in different deployment envitonments.

Advantages :  
High Availability(no Downtime)  
Scalability  
Disaster Recovery(Backups)

  

## Kubernetes Components

- Node : a simple server or working machine.

- Pod :
    
    - Smallest unit of K8. It creates an env the container can run.
    
    - It is an abstraction over containers so that K8 can interact with different containers of different types.
    
    - Usually pod is used to run a single container.
    
    - Each pod has its own IP which is used for communication.
    
    - If a container crashes, the pod is closed and a new pod takes its place. This pod has a new IP which makes it difficult to maintain communications. Hence we use **_SERVICE_**
    

- Service :
    
    - Each pod will have its own service which has its own IP.
    
    - If a pod dies, the new pod created will also hve the same service.
        
        - External Service ⇒ used for external communicationd. Eg- from a web browser.
        
        - Internal Service ⇒ used for communications on a internal level. Eg - b/w a DB pod and other pods
        
    

- Ingress :
    
    - It is used for modification of the urls for external applications.
    
    - Generally urls look like [https://127.43.8.0:8080](https://127.43.8.0:8080), we wantit to be like httpd://myapp.
    

- ConfigMap
    
    - Generally if you want to change the env vars of app ⇒ rebuild
    
    - Config map stores these variables and passes them onto the app. This allows editing without rebuilding. It is not used to store usernames and passwords though.
    

- Secret
    
    - Config map for passwords and usernames.
    
    - Data stored in base64 encoded format.
    

- Volumes
    
    - Attaches a physical storage to a podsothat the data on it is not lost on restart.
    
    - It can be local or remote storage.
    

- Deployments
    
    - In order to avoid downtime when a pod crashes due to updates/internal failure, we create replicas of the pod on multiple nodes.
    
    - Any request is redirected to the least busy pod. This is done by the service.
    
    - These replicas are created by creating a blueprint of the pod and speifying how many to run. This is called a deployment. This is a layer of abstraction over the pods.
    
    - Pods of DB cannot be replicated ⇒ data inconsistency due to multiple entries at the same time. ⇒ Stateful Set
    

- Stateful Sets
    
    - Apps like DB must be replicated using Stateful Sets. This helps in syncronising the entry requests.
    
    - Working with these is very tedious hance we generally deploy stateless applications in K8 clusters and setup databases externally.
    
      
    

# K8 Clusters

### Worker Machines (Nodes)

- Each node runs multiple pods

- Each node must have 3 processes installed
    
    - **container runtime :** to run the container inside the pods. Eg- Docker.
    
    - **kubelet :** interacts with the container and node. Starts the pods and allocates resources from the node to them.
    
    - **Kube Proxy :** Responsible for making the communications happen in a way that optimizes the performance by forwarding requests to least busy pods.
    

### Master Machines (Nodes)

There are multiple master nodes to ensure load balance to the API server and distributed storage for etcd

There are 4 services running in the master machinies

- **API Server :**
    
    - Handles the deployment of new pod sand all the request made between them. Itgets the initial requests of any updates/queries into the cluster.
    
    - Responsible for validation of requests. It is the single entry point to the cluster.
    

- **Scheduler :**
    
    - API server validates any pod creation requests and passes the to the scheduler.
    
    - It decides where the new pod should be scheduled based on the resources available.
    
    - After deciding the node, it makes the kubelet start the pod.
    

- **Controller Manager :**
    
    - Responsible for checking for crached pods. Then passes it to the Scheduler.
    

- **etcd :**
    
    - Stores all the changes made in the cluster in a key value pair format.
    
    - It is the brain of the cluster which guides all the other processes.
    

  

## Minikube

- Setting up an K8 environment is very difficult and resource intensive.

- Minicube lets you have a master node and a worker node on your laptop.

- THis is done using a Virtual Box and used fro testing purposes.

  

## Kubectl

- It is a command line tool that helps us interact with the API server of the K8 cluster

- There are also UI and API interfaces but this is the most powerful.

### install hyperhit(only for mac) and minikube

`brew update`

`brew install hyperkit`

`brew install minikube`

`kubectl`

`minikube`

### create minikube cluster

`minikube start --vm-driver=hyperkit` / `minikube start`

`kubectl get nodes`

`minikube status`

`kubectl version`

### delete cluster and restart in debug mode

`minikube delete`

`minikube start --v=7 --alsologtostderr`

`minikube status`

### kubectl commands

`kubectl get nodes`

`kubectl get pod`

`kubectl get services`

### Creating Deployments

Deployments ⇒ Replicasets ⇒ Pods

Pods cannot be created directly. Instead we create deployments.

`kubectl create deployment __deploymentName --image=__imageName`⇒creates deployment  
⇒the pod created can take some time to create.

`kubectl get deployment`⇒ get all deployments

`kubectl get replicaset` ⇒ get the replica of the deployment.  
podname ⇒ deplymnetName-replicasetId-podId

`kubectl edit deployment nginx-depl` ⇒ opens up the deployment in CLI where it can be edited.

### debugging

`kubectl logs {pod-name}`

`kubectl exec -it {pod-name} -- bin/bash`

### delete deplyoment

`kubectl delete deployment _deplName`

### create mongo deployment

`kubectl create deployment mongo-depl --image=mongo`

`kubectl logs mongo-depl-{pod-name}`

`kubectl describe pod mongo-depl-{pod-name}`

### create or edit config file

`vim _deplConfig.yaml`

`kubectl apply -f _deplConfig.yaml`

delete with config

`kubectl delete -f nginx-deployment.yaml`

\#Metrics

`kubectl top` The kubectl top command returns current CPU and memory usage for a cluster’s pods or nodes, or for a particular pod or node if specified.

  

## Kubernetes Config File

`kubectl apply -f _filePath`

`kubectl delete -f _filePath`

Used to create deployments and services.

Each config file has 3 parts:

- Declaration ⇒ which api to use(fixed for each type), type(depl ,services etc)

- metadata

- specifications ⇒ it is the optimal state that the depl/service must have

- status ⇒  
    it is the current state that the system is in. Kubernetes always tries to match the specs with the current state. This leads to self healing(Suppose a pod crashes and number of replicas is reduced⇒ new podto match specs). Thisis provided by the **etcd**

```JSON
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
        image: nginx:1.16
        ports:
        - containerPort: 8080
```

```JSON
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

Store these files with the code. Use yaml validator to debug.

> [!important] Generally deployment specs have template which has another config to setup the pods.

  

_**Labels and Selectors:**_

Labels are used in the following way

1. Services have a selector which matches with the label of the deployments. This helps them too figure out which deployment to send the request to.

1. Deployments have a selector which matches with the label of the pods. This helps them too figure out which pods to send the request to.

  

_**Ports :**_

This config needs to be done in the service.

It sets which port the service is running and also the targetPort to which it must forward the request to. We have also specified ports for each of the pods that we deploy.

  

`kubectl describe service _serviceName` ⇒ describes a service

`kubectl get pod -o wide` ⇒ get more info on pods

`kubectl get deployment nginx-deployment -o yaml > nginx-deployment-res.yaml` ⇒ get cur config file

  

## Mongo Demo

Use base64 encoded values when creating secrets

`echo -n 'string' | openssl base64`

  

## Namespaces

A way to orgaize resources within a cluster. It is like a cluster within a cluster.

`kubectl get namespace`

By default there are 4 clusters in kubernetes cluster

- kube-system ⇒Used for system processes.Dont use this.

- kube-public⇒ publically accessible data , configmapwith cluster info.

- kube-node-lease ⇒ Stores heartbeat of nodes.Determines availability of nodes

- default ⇒ Resources are located here.

You can also create custoom namespaces.

`kubectl create namespace __name` or use a .yaml file.

### Uses of namespace:

- Used for complex apps to segregate their deployments.

- Used to seperate deployments for different teams. Each team has their own namespace. THis also helps teams to distribute the resources available.

- If dev and staging are both deployed on the same system. they can be segregated. Must share resources(forms 3rd namespace)

- Blue/Green Dev ⇒ Two versions of prod, in the same machine. Must share resources(forms 3rd namespace)

  

### Characterstics of namespace

- Configmaps and secrets are not dhared b/w namespaces.

- Services can be shared across name spaces by providing the containerPorts

- Some resources cannot be created in the namespace. Eg - volume and node
    
    - `kubectl api-resources —namespace=true/false` ⇒ toshow resources in the namespace(true)
    

- `kubectl apply -f __filepath —namespace=_name` ⇒creation in namespace  
    Or specify namespace in metadata

```JSON
apiVersion: v1
kind: ConfigMap
metadata:
	name: mysql-configmap
	namespace: my-namespace
data:
	db_url: mysql-service.database
```

  

## Ingress

Helps to convert the external request to be directed through a url provided by the developer and not the external ip: port we get from minikube.

It replaces the external service and creates its own type of service.

### Ingress Controller

Evaluates all the rules, manages redirections and is the entry point to the cluster

K8 Nginx Ingress Controller

You need a load balancer to redirect requests to the controller. It will be a node with a public IP, ports which will act as an entry point to the cluster.

`minikube addons enable ingress` ⇒ installs ingress

`kubectl get pod -n kube-system` ⇒ to get ingress pod data

```YAML
apiVersion: networking.k8s.io/v1betal
kind: Ingress
metadata:
	name: dashboard-ingress
	namespace: kubernetes-dashboard
	spec:
		rules: \#defines the host(ingress) address and the address all requestmust be forwarded to
			- host: dashboard.com \#must be a valid domain name, ?Map it to the node's IP
			http:
				paths:
					- backend:
							serviceName: kubernetes-dashboard \#internal service name
							servicePort: 80 \#internal service port
```

`kubectl get ingress -n kubernetes-dashboard`

This creates an ingress. After its external IP is assigned,  
`sudo vim /etc/hosts` ⇒ add  
`__externalIP` [`dashboard.com`](http://dashboard.com) ⇒ maps the address

  

_**Default Backend ⇒**_ Used to handle calls to unmaped address to the external server.

### Uses :

Multipaths

```YAML
apiVersion: networking.k8s.io/v1betal
kind: Ingress
metadata:
name: simple-fanout-example
annotations:
nginx.ingress.kubernetes.io/rewrite-target: /
spec:
rules:
host: myapp.com
http:
paths:
path: /analytics
Баскепа:
serviceName: analytics-service
servicePort: 3000
path:
/shopping
backend:
serviceName: shopping-service
servicePort: 8080
```

multi sub-domains

```YAML
apiVersion: networking.k8s.io/v1betal
kind: Ingress
metadata:
name: name-virtual-host-ingress
spec:
rules:
- host: analytics.myapp.com
http:
paths:
backend:
serviceName:
analytics-service servicePort: 3000
host: shopping.myapp.com
http:
paths:
backend:
serviceName: shopping-service
servicePort: 8080
```

Getting https // ⇒ TLS cert ⇒need a secret

![[/1 2.png|1 2.png]]

  

- Helm

- It is a package manager for K8

- It contains basic Yaml files for deploying various components/services of K8.

- Helm Charts ⇒ bundles of yaml files for setting up services.

- `helm search <keyword>` or Helm Hub

- It also acts as a templating engine.

```YAML
\#YAML config
apiVersion: v1
kind: Pod
metadata:
	name: {{ .Values.name}}
spec:
	containers:
	- name: {{ .Values.container.name }}
		image: {{ .Values.container.image }}
		port: {{ .Values.container.port }}
```

```YAML
\#values.yaml
name: my-app
container:
	name: my-app-container
	image: my-app:1.0
	port: 9001
```

_**Helm Chart Structure**_  
mychart/  
Chart.yaml ⇒ meta info about chart  
values.yaml ⇒ values for templates, with default values for the chart set by publisher  
charts/ ⇒ dependencies of the chart are listed here  
templates/ ⇒ template files for the deployment  
`helm install <chart_name>`

  

You can also add new values to the chart by using

1. Define your own subset of variables you want to change ina new yaml file  
    `helm install --values=my-values.yaml <chart_name>`

1. `Use —set var_name = value`

  

_**Tiller**_ ⇒ It is a server that maintains the list of all installs/changes made via charts

helm install chartName  
helm update chartName —values=values.yaml  
helm rollback chartName

Tiller has too much power hence they removedit in v3

  

## Volumes

1. Storage must not depend on pod lifecycle(Data persistence)

1. Storage must be available to all nodes.

1. Storage must survive even if thecluster fails.

  

### Persistent Volume:

- It is a cluster storage defined using a yaml file.

- Takes the storage externally and is not resposible for data. The cluster admin is responsible.

- _**Persistent Volume Claims ⇒**_ how pods claim volumes, required that both exist in the same namespace  
    

![[1 1.png]]

  

> [!important] ConfigMaps and Secrets are also volumes

This is how you mount volume into the container

![[/1 2 2.png|1 2 2.png]]

### Storage Class

It provisions Persistent Volumes whenever a PVC Claim is claimed.

It can also be created using yaml files.

  

## Stateful Sets

Used for deployment of stateful apps like DBs.

Comparision with deployments

- Pods cant be created/deleted at the same time.

- pods cant be randomly accessed

- replica pods are not identical and have a sticky identity which is podName-0,podName-1, podName-2 …
    
    - ⇒ To prevent data inconsistency all pods cannot be treated as same as this will allow multiple users to change the same data.
    
    - In fact only one pod is allowed to write(MASTER POD), while reading can be done by any pod(SLAVE PODS).
    

- All pods have different memory allocation and need to constatntly sync with each other.

- Each pod has its state which get reattached to the new pod when the old pod dies.This is done using a remote storage.

- First pod is the master and all pods are created in order. If any creation fails the pod next in line are not created as well. Similary deletion starts from the end.

## Services

- provides a stable IP as pods are always dying and new pods have new IPs as well.

- Load balancing

Services have a selector to match the pods to forward the request to. They also have a target pod to select the container.  
Service may have multiple ports open to allow multi app requests.

_**Headles Service ⇒**_ used for communicating with a particular pod, using the DNS Lookup.

Service Types:

- ClusterIP ⇒ regular service

- NodePort ⇒ Opens a port on each worker node to allow external requests. , port ranges in 30000 and 32767. It is not secure as the user has access to node.

- LoadBalancer ⇒ External Service. Combination of ClusterIP and NodePort service. Here there is no access to nodes, instead all request go through the internal service tovarious nodes.