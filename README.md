# Kubernetes

### Table of content
+ [What is kubernetes?](#what-is-kubernetes-)
+ [Components of kubernetes:](#components-of-kubernetes-)
+ [Installation](#installation)
+ [Basic Commands](#basic-commands)
+ [K8 Configuration YAML files](#k8-configuration-yaml-files)
+ [Kubernetes Architecture](#kubernetes-architecture)
+ [Kubernetes Namespace](#kubernetes-namespace)
+ [Kubernetes Ingres](#kubernetes-ingres)
+ [Kubernetes Volumes:](#kubernetes-volumes-)
+ [Statefulset](#statefulset)
+ [Kubernetes Service:](#kubernetes-service-)
### What is kubernetes?
* Container orchestration platform
* Benefits of Kubernetes:
    * **High availability** - Low down time
    * **Scalability** – High performance
    * **Disastor recovery** – Backup and restore

### Components of kubernetes:
<img src="image\k8-architecture.PNG" width="500" height="300">

* **Nodes**
    * Nodes are and virtual or a physical machine in a K8 cluster
* **Pods**
    * Smallest unit K8
    * An abstraction over a cotainer
    * Usually one application per pod
    * Each pod gets it's own IP address
    * A new IP address on recreation
* **Service**
    * Permanent IP address
    * Life cycle of pod and service are not connected
* **Ingress**
    * Used to route traffic inside K8 cluster.
    * Ingress is for a proper url for your appliation, any request will go  to ingress then it will be mapped to the service
* **ConfigMap**
    * External configuratio for your application
* **Secret**
    * Used to store secret data (For example database username and passwords)
    * Base64 encoded format
* **Volumes**
    * Data storage
    * Can be a storage on local disk or on a cloud server outside k8
* **Deployment**
    * Blueprint of an application pod
    * In real life we will be working with deployments not pods where we will specify how many replicas we want to create
    * Abstraction of pod
* **Statefulsets**
    * DB can't be replicated using deployments because DB contains a state which is it's data
    * All the replicas should access the same database and which replica is writing the data and which one is reading should be monitored which is not possible using deployments.
    * Deployment is for stateless application and Statefulset is for stateful application or databases

**Note:** Deploying databases using statefulset is not easy that is why it's best practice to deploy databases outside K8 cluster

### Installation
* Minikube: Provides a single node kubernetes cluster in local system which can be used for testing or learning purpose.[Install Minikube](https://minikube.sigs.k8s.io/docs/start/)
* kubectl: Command Line Interface which is used to interact with the cluster. [Install kubectl](https://kubernetes.io/docs/tasks/tools/)

* Note: Make sure in the system virtualization is enabled, as Minikube requires some Hyper-V or Docker.

### Basic Commands
Get all the nodes in your cluster
```
kubectl get nodes
```
Get the pods available in the K8 cluster
```
kubectl get pods
```
Like this you can get the list of Deployments, Services etc.
```
kubectl get deployments
```
Create a deployment
```
kubectl create deployment NAME --image=image -- [COMMAND] [args...] [options]
```
For example in the following command I have deployed a nginx container
```
kubectl create deployment nginx-demo --image=nginx
```
The deployment may take a while. You can check the pod status using
```
kubectl get pods
```
Getting extra information from pods
```
kubectl get pods -o wide
```
In kubernetes all the components you can create using YAML file. You can use your own YAML file or you can edit the configuration as YAML as well.
For example for changing some configuration in nginx deployment
```
kubectl edit deployment nginx-demo
```
For getting pod logs or for pod description
```
kubectl logs <POD NAME>
kubectl describe pod <POD NAME>
```
As pod is just an abstraction on top of container, you can basically enter inside the container to test configuration
```
 kubectl exec -it <POD NAME> -- bin/bash
```
### K8 Configuration YAML files
* There are 3 parts for each configuration files
	* Metadata
	* Specification
	* Status (Created automatically)

Following are the examples of Deployment and Service
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: <Image>
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: <Port>
```
```
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: <Port>
    targetPort: <Target Port>
```
* Status is basically the current state which is basically compared with Specs and based on that kubernetes do the selfhealing.
* Status part is generated from 'etcd'
* Template:
	* Has it's own metadata and sepcs
	* Applies to pod
	* Blueprint for the pod
* Selectors and labels
	* Metadata parts contain the label
	* Spec contains selector
* Note:You can create your own components as YAML file then use the following command
```
kubectl apply -f <FILE NAME>
``` 
### Kubernetes Architecture
* Node Processes
    * Each node has multiple pods on it
    * Processes must be installed on each node
	    * **Container runtime**
        * **Kubelets** – Interact with both container and node. Starts a pod with a container inside.
		* Communication via services – Load balancer
		* **Kube proxy** – Make sure communication happens between the pods in a proper way.

* Master Node:
    * 4 processes should run on every master node.
        * **API server** (cluster gateway)
        Some request -> API server -> Validate request -> Other processes
        * **Scheduler**:
        Schedule new pod -> Scheduler ->Where to put new pod
        Scheduler decides on which node a new pod should be  scheduled
        * **Controller manager**: Detects state changes
        * **Etcd**: Cluster brain. Cluster changes get stored in the key value store Application data is not stored.

    * **Note**: Master node requires less resources than worker nodes.

### Kubernetes Namespace
* Organise resources in namespaces
* Virtual cluster inside a cluster
* By default kubernetes provides four namespaces:
	* Kubernetes-dashboard: Only with minikube
	* Kube-system
		* DO NOT create or modify kube-system
		* System proesses
		* Master and kubectl prcesses
	* Kube-public
		* Publicly accessible data
		* A configMap which contains cluster information
	* Kube-node-lease
		* Heartbeats of node
		* Each node has associated lease object in namespace
		* Determines the availability of the node
	* Default
		* Resources you create are located here
* Why to use namespaces:
	* Resources grouped in namespaces
	* Should not create a namespace for smaller projects
	* Still it's good idea to create namespaces for each project
	* If different teams uses same cluster it's better to use separate namespaces
	* Resource sharing: staging and development
	* Access and resource limit on namespaces, per namespace one can define resource quota
* Summary of use cases of namespaces:
	* Stricture your components
	* Avoid onflicts between teams
	* Share services between different environments
	* Access and resource limits on namespace level
* Characteristics of namespaces:
	* Cannot access most resources and another namespace
	* Can access service from another namespace
	* Components which cannot be created in namespace. Example: volume and node

### Kubernetes Ingres
* We can use external service to access the application however this is useful for testing purpose. 
* In the final product it should contain an ingress which then forward request to the internal service
* Routing rules:
	* Forward request to the internal service
* How to configure ingess in kubernetes cluster:
	* Need an implementation for ingress which is ingress controller
* What is Ingress controller
	* Evaluate all the rules
	* Manages redirection
	* Entry point to the cluster
* Environment on which your cluster runs:
	* In Cloud providers they have cloud load balancer which redirects the request to ingress controller.
	* As bare metal, general practice is to use proxy server, so the request will be forwarded to proxy server then it will go to ingress controller.
* Install Ingress controller in Minikube
	* Minikube addons enable ingress
	* It will automatically starts k8 Nginx implementation of ingress controller

### Kubernetes Volumes:
* Storage requirements:
	* Storage that doesn't depend on pod life cycle
	* Storage must be available to all nodes
	* Storage needs to survive even if the cluster crashes
* Persistent volume:
	* A cluster resource
	* Created via YAML files
	* Needs actual physical storage, like local disk, nfs storage, cloud storage(AWS S3)
	* You have to decide:
		* What type of storage you need
		* Create and manage them  by yourself
		* Depending on the storage type or storage backend some of the attributes in the YAML differs
	* Persistent volume is not under any namespace
	* Accessible to the whole cluster
* Local vs Remote volume:
	* Each volume has it's own use cases
	* Local volume violates requirements for data persistance:
		* Being tied to a specific node
		* Surviving in cluster crashes
	* Because of these scenarios for database remote volumes are used.
* Persistent volume claim
	* Application has to claim persistent volume
	* Pod requests the volume through the PV claim
	* Claim tries to find a volume in cluster
	* Volume has actual storage backend
	* Claims must be in the same namespace
* Special volumes:
	* ConfigMap and Secret:
		* Local volumes
		* Not created via PV or PVC
		* Managed by K8
* Note: A pod can use different types of pods simultaneously
* Storage Class:
	* Provisions PV dynamically whenever PVC claims it
	* Created using YAML file
	* Storage backend is defined in Storage Class component via "provisioner" attribute

### Statefulset
* K8 component used for stateful application like databases
* Stateful applications are all databases or any application that stores data to keep track of it's state.
* Stateless applications don't keep record of state
* Each request is completely new
* Deployment of stateless and stateful applications
	* Stateless applications deployed using deployments
	* Stateful applications deployed using statefulset
	* Both manage pods based on container specifications
	* Configure storage the same way for both
* Deployment vs Statefulset
	* Deployment
		* In deployment the replicas are identical and interchangeable
		* Created in random order and with random hash
		* One service that load balances to any pod
	* StatefulSet
		* Can't be created and deleted at the same time
		* Can't be randomly addressed
		* Replica pods are not identical:
			* Pod identity: sticky identity for each pod
			* Created from same specification but not interchangeable
			* Persistent identifier across rescheduling
	* Why this identity is necessary?
		* Scaling database applicaions:
			* There will be one master node which can write and read data at the same time
			* Then there will be workers or slaves which can only read the data
			* These pods don't use same physical storage, so for maintaining the same data the pods need to continuously synchronize the data
	* In statefulSet it's good practice to use persistent volume (Remote volume)
	* So that data survives even if all the pod die
	* Pod state
		* All the pods have their own state for example whether it is a master or a slave and the state information is stored in the pod's persistent storage
		* That's why it's good to use remote storage so even if the pod gets rescheduled the pod state will remain.
    * Pod identity
        * Where deployment pods have random hashes as pod identifier.
        * statefulSet pods have fixed identifiers which consists of $(StatefulSet name)-$(ordinal)
    * Pod endpoints
        * Like deployment StatefulSet contains it's load balancer which refers to all the pods
        * Also each pod has its individual service name
* Charataristics
    * Predictable pod name
    * Fixed individual DNS names
    * When pod restrats:
        * IP address changes
        * Name and endpoints stays the same
    * Sticky identity make sure that the pods restored its state and role
* **Note**: Stateful applications are not good candidate for containerized environment

### Kubernetes Service:
* Each pod has its own IP address
	* Pods are ephemeral - are destroyed frequently
* Service
	* Stable IP address
	* Load balancing
	* Loose coupling
	* Within or outside cluster
* Types:
	* ClusterIP
		* Default type
		* Internal service
		* Browser will request to ingress which in turn hand over the request to internal service which is an abstraction over an IP address
		* Service communication: selector
			* Pods are identified via selector
			* Key value pairs
			* Labels of pods
			* Random label name
			* SVC matches all 3 replicas
			* Registers all the pods
			* Must match all the selectors
		* Pods with multiple ports:
			* In this case target port defined which port to refer
		* When a service is created k8 creates a service endpoint which keeps track the pods which are members/endpoints of the service
        * Note:
            * Service port is arbitrary
            * Target port must match the port of the pods
	* Headless
		* Clients want to communicate with one specific pod
		* Pods want to talk directly with one specific pod
		* Use cases: stateful applications like databases
		* Client need to figure out the IP address
			* Option 1 API call to k8 server
				* Inefficient
				* Makes app too tied with k8 APIs
			* Option 2 DNS lookup
				* It will give the IP address for the service
				* Keep clusterIP attribute as None in the service configuration then it will return the pods IP
* Service type attributes
	* ClusterIP: default one
	* Nodeport: 
		* The clusterIP is accessible within the cluster
		* NodePort allows external traffic to access a fixed port in each worker node.
		* NodePort has range: 30000 - 32767
		* NodePort service is not secure
	* LoadBalancer:
		* Extension of NodePort service
		* NodePort service is an extension of ClusterIP type

* Refernces
    * https://www.youtube.com/watch?v=X48VuDVv0do
    * https://kubernetes.io/docs/home/
    * https://www.youtube.com/watch?v=Wf2eSG3owoA&t=40s
