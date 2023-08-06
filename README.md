kubernetes: usually its a cluster of multiple nnode
  1. master node - All the managing process  are done by master node
  				four processes run on every master node that control the cluster state and worker nopde
  					1. APi server - cluster gateway which get the initial requests, act as gatekeeper for authentication
  							eg:  some req -> api server -> validate req -> other process
  					2. Sheduler -  its shedule on which node(worker) next pod to put or sheduled( based on resource calculation)
  					3. controller manager - detect cluster state change, it will control when any pod die then it will ask shedule to shedule pod or rerstart
  					4. etcd - its a key value store of cluster state, its a cluster brain

  2. worker node
	---------------------
	|					|
	|	app		--		|
	|				ser	|
	|		---		vic	|
	|	db 		--	es	|
	|					|
	|					|
	---------------------

 components:
  Pod - abstraction layer
  Services m- communication between pods
  ingress - routing
  config - db details
  secret - confidential part
  database(volume)
  Deployment
  Stateful set - for stateful application like database

  Node: - Node are rthe cluster srervice, Each node has multiple pod on it
  			Three things to be installed on every node:

  				1.  container runtimwe
  				2. kublet - proces off k8s, it interact with both the container andf the node, its start tyheb pod with container inside
  				3. kube proxy - make sure that communication also works in performant wway with low overhead

Minikube - 1 nod3e k8s cluster that runs on virtual box
 minikube: a ONE Node cluster, where the master and worker processes are on the same machine. 
Kubectl, the command line tool for Kubernetes, then enables the interaction with the cluster: to create pods, services and other components.

deployment : -  kubectl create deployment nginx-depl --image=nginx
			blueprint of creating pods, which manage replicaset
Replicaset:- managing replica of pod

CRUD commands:
			kubectl create deployment name
			kubectl edit deployment name
			kubectl delete deployment name
Status:
			kubectl get nodes|pod| services|replicaset|deployment
Debugiing pods:
			kubects logs podname
			kubectl describe pod podname
			kubectl exec -it pod name --bin/bash
use configuration file for CRUD:
			kubectl apply -f filename -> Apply a configuration file
			kubectl delete -f filename -> Delete a configuration file


kubectl apply -f nginx-deployment.yaml - to create deployment file
kubectl apply -f nginx-service.yaml -- to apply service configuration

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
        - containerPort: 80


      -------------------
Eack configuration file has theee parts:
1. metadata
2. specification
3. status => Desire state = Actual state

deployment.yaml
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

service.yaml
			  apiVersion: v1
				kind: Service
				metadata:
				  name: nginx-service
				spec:
				  selector:                              => to connect to pod through label
				  	app: nginx
				  ports:
				  	- protocol: TCP
				  		port: 80  												=> service port
				  		targetPort: 8080  						`		=> container port for deployment

		Db service
		    |
		    |port : 80
		nginx service
				|
				|targetPort:8080 (service need to know on which pod it should forward the request)
			Pod 

kubectl get deployment nginx-deployment -o yaml
kubectl get deployment nginx-deployment -o yaml > nginx-deployment-result.yaml  => put all things in nginx-deployment-result.yaml   
kubectl get pod -o wide --> to get additional output      

echo -n 'username' | base64 -> to encrypt

The command set kubectl apply is used at a terminal's command-line window to create or modify Kubernetes resources defined in a manifest file. This is called a declarative usage. The state of the resource is declared in the manifest file, then kubectl apply is used to implement that state.    

type: loadbalancer=> assign service an external ip address and so acceptsezternal requests
internal service and type cluster ip is default

nodePort => port for external ip address open, it has some range between 30000 to 32767   
 
 ----
 when we create database on http://172.21.0.2:30000/
 process on background occur like below
change on mongo express -> mongo express external service => mongo express pod => mongo db internal service => mongo db pod       


-------------------------------

Namespace: virtual cluster inside a cluster

kubectl get namespace
NAME              STATUS   AGE
default           Active   3d5h  => resources you created are located here
kube-system       Active   3d5h  -> dont create and modify in kubesaystem
kube-public       Active   3d5h	 -> publicaly cess data
kube-node-lease   Active   3d5h  => heartbeet of node , each node has associated leaser objectin name space, determine the availibility of node    

create namespace:
	kubectl create namespace name

Also we can create namespace using configuration file

apiversion: v1
kind: ConfigMap
metadata:
	name: mysql-configmap
	namespace: my-namespace
data:
	db_url: mysql-service.database

why we need namespace:
		help to improve performance of given cluster.
		if a cluster is separated into miltiple namespace for different projects, the k3s api will have fewer items to search for performing operations

		use cases:

			1. structure your components
			2. avoid conflicts between team
			3. share service between differentn environment
			4. access and resource limit on namespace level

			kubectl create namespace my-namespace  => to create namespace 


			create config in specific namesopace:
				kubectl apply -f filename --namespace=my-namespace


-------------------Ingress----------
		An API object that manages external access to the services in a cluster, typically HTTP.
		Ingress may provide load balancing, SSL termination and name-based virtual hosting.

		ingress controller=> to evaluate all rule which we define in thge cluster and manage all redirections

		install ingress controller

		ingress configuration:
			apiVersion: networking.k8s.io/v1betal\
			kind: Ingress
			metadata: 
				name: myapp-ingress
			spec:
				rules:
				- host: myapp.com  		 => valid domain addrress
					http: 							 => incoming request get forwarded to internal service
						paths:	           => url path
						- backend: 
								serviceName: myapp-internal-service
								servicePort: 8080


								use tls for https


	-------------------Helm ----------------

		package manager for kubernetes, tom package yaml files and distribute them in public and private repositorrries.

		bundle of different yaml  file like service , secret, config is called helm chart

		Kubernetes applications packages are structured in the Helm packaging format called charts.

		helm chart structure:
		mychart/           => name of chart
			chart.yaml       => meta info about chart
			values.yaml      => values for template file
			charts/          => chart dependencies
			templates/       => actual template file

	cmds:
	helm install <chartname>

	eg:
	values.yaml:
		imageName: myapp
		port: 8080
		version: 1.0.0
	my-values.yaml:
		version: 2.0.0

  After running: helm install --values= my-values.yaml <chartname>

  result will be like:
  imageName: myapp
  port: 880
  version: 2.0.0

  OR 
  helm install --set version=2.0.0

--------------k8s volume--------------

		three component of k8s storage:
			1. persistent volume(yaml file is on 2:44 min)
			2. persistent volume claim (2:48)
			3. storage class => it provisi0n persistant volume dynamically (2: 56, 58)

		pod request the volume through PV claim.  => claim tries to find volume in cluster => volume has the actual storage backend

		pv claim must be in same namespace

		pod claim storage via pvc => pvc request storage from sc => sc creates pv that meets the need of claim


		------------------stateful set ---------------

		stateful application: databases like mysql, mongo, elastic search
			Any application that stores data to keep track of its state orr that application that track states by saving the information in some storage
			stateful appl;ication deployed using statedful set

			stateful with three replicas:
				next pod is only created when previous is up and running
				deletion in reverse order , starting from theb last one

				predictable pod name
				fixed invidual dns service\
				when pod restart  ip address changes but name and endpoint stay same

		stateless application: dont kep record of state, each request is completely new and isolated
			stateless application dfeployed using deployment



			----------------kubernetes services--------------

			each pod has its own ip address

				difference service tpe:
					clusterip service, -- connect pod with service

					headlesss service, -- client want to communicate with one specific pod directly
							client need to figure out ip address of each pod
								dns lookup for servioce -> return single ip address(cluster ip), set cluster ip to none to return pod ip address instead
								no cluster ip addrress is assigned

			   	nodeport servicr,
					loadbalancer se4rvice


			service type attributye:
				clusterip => clusterrip accesible only within cluster, internal service


				nodeport  -> create a service that is accesible on static port, external traffic has accessed to fix port on each worker node, its not secure
											it is an extension of clusterip service


				loadbalancer -> become accessible externally through cloud provider loadbalancer
												nodeport and cluster ip created automatically by kubernetes eg: aws, azure, gcp
												it is an extension opf nbodeport type service



		------------------Ingress------------------

					Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. 
					Traffic routing is controlled by rules defined on the Ingress resource.

					 Ingress frequently uses annotations to configure some options depending on the Ingress controller


					 CRD  => custom code that define a resource to add your kubernetes API server without building a complete custom server.




---------------- cmds------------------\

				kubectl proxt  => to startiong the serve4hsahas


----------  cmds ----------------
send file from local to mount path off container:
		kubectl cp data.txt my-deployment-9b9579b64-znwhv:/path/mount

create file in mount path of container:
		kubectl exec my-deployment-9b9579b64-znwhv -c my-container -- sh -c 'echo "Hello Piyush" > /path/mount/myfile.txt'

list of file in mount path of container:
		kubectl exec my-deployment-9b9579b64-znwhv my-container -- ls /path/mount
		kubectl exec -it my-deployment-9b9579b64-znwhv -- ls /path/mount

copy file from mount path of container tyo local:
		kubectl cp my-deployment-9b9579b64-znwhv:/path/mount/data.txt xyz.txt

in all above scenario: my-deployment-9b9579b64-znwhv is pod name