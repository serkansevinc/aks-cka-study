## UNDERSTANDING KUBERNETES

This repo is for anyone who wants to appear for CKA(https://www.cncf.io/certification/cka/) exam and wants to know the basics and practice. I have created this repo while I am preparing for the exam to help other fellow engineers like me.

The folders in the repository contain the YAML files which contain the definition of most of the k8s objects like <i>Pods, services, replication controller etc</i>.

<b>Best course out there to practice and learn k8s fundamentals and prepare for the exam: https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/learn </b>


### IMPORTANT EXAM RESOURCES
<b>
  
 1) K8s command Cheatsheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
  
2) Free Online k8s playground: https://www.katacoda.com/courses/kubernetes/playground
  
3) Practice tests : https://kodekloud.com/topic/practice-tests-deployments-2/
  
4) Using k8s Imperative commands to create Objects: https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/ 

</b>



### EXAM TIPS


As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. 

During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. 

Using the ``` kubectl run``` and ``` kubectl create <object>``` command can help in generating a YAML template. And sometimes, you can even get away with just the ```kubectl run``` command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with specific name and image you can simply run the ```kubectl run``` or ``` kubectl create``` command.

Use the below set of commands and try the previous practice tests again, but this time try to use the below commands instead of YAML files. Try to use these as much as you can going forward in all exercises


```--dry-run```: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the ```--dry-run=client``` option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.

```-o yaml```: This will output the resource definition in YAML format on screen.



Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

Reference (Bookmark this page for exam. It will be very handy):

```yaml

https://kubernetes.io/docs/reference/kubectl/conventions/

```



### A quick note on editing PODs and Deployments

#### Edit a POD


Remember, you CANNOT edit specifications of an existing POD other than the below.

```yaml 

spec.containers[*].image

```

```yaml 

spec.initContainers[*].image 

```

```yaml 
spec.activeDeadlineSeconds

```

```yaml

yaml spec.tolerations

```

For example you cannot edit the environment variables, service accounts, resource limits of a running pod. But if you really want to, you have 2 options:

1. Run the ```kubectl edit pod <pod name> ``` command.  This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.



A copy of the file with your changes is saved in a temporary location.

You can then delete the existing pod by running the command or use ```kubectl replace --force``` command as well instead of creating a new pod:

```yaml 
kubectl delete pod webapp
```

Then, create or replace the existing pod with new changes:

```yaml

kubectl replace --force -f /tmp/kubectl-edit-ccvrq.yaml

```
OR

Then create a new pod with your changes using the temporary file

```yaml
kubectl create -f /tmp/kubectl-edit-ccvrq.yaml
```



2. The second option is to extract the pod definition in YAML format to a file using the command

```yaml 
kubectl get pod webapp -o yaml > my-new-pod.yaml
```

Then make the changes to the exported file using an editor (vi editor). Save the changes

```yaml
vi my-new-pod.yaml
```

Then delete the existing pod

```yaml
kubectl delete pod webapp
```

Then create a new pod with the edited file

```yaml
kubectl create -f my-new-pod.yaml
```



#### Edit Deployments

With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

```yaml
kubectl edit deployment my-deployment
```




## IMPORTANT ```kubectl``` COMMANDS



### Create an NGINX Pod

```yaml
kubectl run nginx --image=nginx
```

```yaml
kubectl run --image=nginx custom-nginx --port=8080
```  

The above command creates a new pod called ```custom-nginx``` using the nginx image and exposes it on container port ```8080```.

### Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

```yaml
kubectl run nginx --image=nginx --dry-run=client -o yaml
```



### Create a deployment

```yaml
kubectl create deployment --image=nginx nginx
```

### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

```yaml
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4) and write it to a file.

```yaml
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

### Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.

```yaml
kubectl create -f nginx-deployment.yaml
```

OR

In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.

```yaml
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```


### Creating a Service and exposing a pod


#### 1) Using ```kubectl create```


```yaml
kubectl create service nodeport <myservicename> 
```

<b>This will not use the pods labels as selectors, instead it will assume selectors as ```app=redis```. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service </b>



#### 2) Using ```kubectl expose```

<b>This will automatically use the pod's labels as selectors</b>

```yaml
kubectl expose pod nginx  --port=80 --target-port=8000 --dry-run=client -o yaml 
```

OR

  ```yaml
  kubectl expose pod httpd --target-port=80 --port=80 -o yaml > service1.yaml
  ```

  then 

  ```yaml
  
  kubectl create -f service1.yaml

```


### EXAMPLES

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

```yaml

kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml

```


Or

```yaml
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 

```

Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

```yaml

kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```
(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

```yaml
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```
(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the ```kubectl expose``` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.



### Selectors and Labels

Lables are properties assigned to each item.  Selectors helps us filter these items.

Labels are added under the ```metadata``` section of the resource definition file, where as selectors are added under the ```spec``` section.

Example:

```yaml

# nginx-pod.yaml
apiVersion: v1
kind: Pod # kind of object we are creating
metadata: 
  name: nginx-pod
  labels:
    app: nginx
    tier: dev
spec: # is a dictionary
  containers: # is a list
  - name: nginx-container # first item in the list  
    image: nginx
    
```

Selectors

```yaml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: replicaset-1
spec:
   replicas: 2
   selector:
      matchLabels:
        tier: nginx # should match the pod label
   template: # pod definition goes here
     metadata:
       labels:
        tier: nginx
     spec:
       containers:
       - name: nginx
         image: nginx
            

```


#### Useful commands

```yaml

kubectl get pod --show-labels

kubectl get pod --selector add=dev # or kubectl get pod -l add=dev

kubectl get pod -l env=prod,bu=finance,tier=frontend  # equality based selectors

kubectl get pod -l 'env in (prod) ,bu in (finance), tier in (frontend)' # set based selectors

```

### Taints and Tolerations

They are used to set restrictions on what pods can be scheduled on a Node. It does not tell a pod to go to a particulr Node, instead it tells the Node to only accept Pods with certain Tolerations to taints. 

Different kinds of Taint effects:

1) NoSchedule - The pod won't be scheduled on this Node.
2) PreferNoSchedule - Try to avoid placing a pod on this node, but it's not guranteed.
3) NoExecute - Schedule no nodes and evict any nodes if any running on the Node.

The Master node has a taint set to it. To check the taint we can use the command:

```yaml

kubectl describe node kubemaster | grep Taint

```

Command to taint a Node:

```yaml

kubectl taint node node01 spray=mortein:NoSchedule

```
Command to remove the taint from the Node. Append a '-' to the end of the ```taint``` command.

```yaml

kubectl taint node node01 spray=mortein:NoSchedule-


```

Adding toleration inside the Pod definition, so that the node can run that Pod:

```yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: bee
spec:
  containers:
   - image: nginx
     name: bee-container

  tolerations:
   - key: "spray"
     value: "mortein"
     operator: "Equal"
     effect: "NoSchedule"

```


### Node Selectors and Node Affinity

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

Node affinity is conceptually similar to nodeSelector, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity:

<b>requiredDuringSchedulingIgnoredDuringExecution</b>: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.


<b>preferredDuringSchedulingIgnoredDuringExecution</b>: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

Note: In the preceding types, <b>IgnoredDuringExecution</b> means that if the node labels change after Kubernetes schedules the Pod, the Pod continues to run.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: blue
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: blue
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: blue
    spec:
      containers:
      - image: nginx
        name: nginx

      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: color
                  operator: In
                  values:
                  - blue
```

Another example of setting Node affinity for a pod using ```Exists``` operator:

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: red
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      app: red
  template:
    metadata:
      labels:
        app: red
    spec:
      containers:
      - image: nginx
        name: nginx

      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: node-role.kubernetes.io/control-plane
                  operator: Exists
                  
         
```



## Daemon Sets and Static Pods

1) <b>Daemon Sets</b>:  ensure that all the nodes across the cluster are running a copy of a Pod. Use cases include deploying a monitoring agent, logging agent, kube-proxy agent as this is require to be run on all Nodes in a k8s cluster.


2) <b>Static Pods</b>: These are pods which are created by ```kubelet``` independently without the intervention of k8s control plane components like scheduler and API server. This happens by placing the pod's manifest inside a directory on a host machine and then adding that configuration/location inside the kubelet service configuration. The kubelet manages these pods, whereas the k8s control plane k8s controller has a read-only access to these pods.

### Use Case of Static Pods:  

Clusters depoyed using ```kubeadm``` & ```minikube``` use this approach to deploy control plane components. Static pods are mainly used to deploy k8s control plane components like ```kube-api-server``` , ```controller-manager```, ```scheduler``` &  ```etcd``` server etc. We can simply install ```kubelet``` on the master Node and then place the Pod menifests inside a directory and then add the location to those manifests inside the ```kubelet``` configuration file.

In general, the kubelet configuration file is inside :

```yaml

/var/lib/kubelet/config.yaml

```

And inside the above configuration we need to add the location of the pod's manifests to:

```yaml

staticPodPath: /etc/k8s/manifests

```

In above case we are assuming the static pod's manifests are inside ```/etc/k8s/manifests``` location on the host. 

<b>NOTE : One way to identify a Static Pod is that, the name of the static pod ends with the name of the Node on which they are running. </b>

### How to identify a Static pod:

1) List the pods and check which all are Static Pods. Generally, control plane components are deployed as Static Pods under ```kube-system``` namespace. In this example we have deployed a ```minikube``` k8s cluster.

```yaml

> kubectl get nodes -o wide 
NAME       STATUS   ROLES           AGE    VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    control-plane   137d   v1.24.1   192.168.49.2   <none>        Ubuntu 20.04.4 LTS   5.10.47-linuxkit   docker://20.10.17

> kubectl get pod -n kube-system
 
NAME                               READY   STATUS    RESTARTS          AGE
coredns-6d4b75cb6d-g6fk8           1/1     Running   8 (9m48s ago)     137d
etcd-minikube                      1/1     Running   8 (9m48s ago)     137d
kube-apiserver-minikube            1/1     Running   8 (9m48s ago)     137d
kube-controller-manager-minikube   1/1     Running   8 (9m48s ago)     137d
kube-proxy-5pmx4                   1/1     Running   8 (9m48s ago)     137d
kube-scheduler-minikube            1/1     Running   8 (9m48s ago)     137d
metrics-server-8595bd7d4c-lk6mh    1/1     Running   237 (9m48s ago)   137d
storage-provisioner                1/1     Running   164 (8m46s ago)   137d
tiller-deploy-c7d76457b-bztz7      1/1     Running   164 (9m48s ago)   137d


```
Now in the above example, a simple way to identiy Static pods is by looking at their names - In the above pods whose names end with minikube are static pods and they are control plane components as well- ```kube-apiserver-minikube ```, ```kube-controller-manager-minikube```, ```kube-scheduler-minikube   ```, ```etcd-minikube```.


2) Then login to the Node and check the configuration file of the ```kubelet``` process:

```bash

ps -ef | grep kubelet
root        1297       1  6 08:47 ?        00:09:24 /var/lib/minikube/binaries/v1.24.1/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=remote --container-runtime-endpoint=/var/run/cri-dockerd.sock --hostname-override=minikube --image-service-endpoint=/var/run/cri-dockerd.sock --kubeconfig=/etc/kubernetes/kubelet.conf --node-ip=192.168.49.2 --runtime-request-timeout=15m

```
From the above we can notice the configuration of kubelet process ```--config=/var/lib/kubelet/config.yaml```

Now, we just need to check for ```staticPodPath``` inside the above file and that will give us the path to the Static pod's manifests:

```bash

sudo cat /var/lib/kubelet/config.yaml | grep -i static
staticPodPath: /etc/kubernetes/manifests

```
Now by navigating to that specific directory we can locate the Static pod's manifests:

```bash

docker@minikube:/var/lib$ cd /etc/kubernetes/manifests
docker@minikube:/etc/kubernetes/manifests$ ls
etcd.yaml            kube-controller-manager.yaml
kube-apiserver.yaml  kube-scheduler.yaml
docker@minikube:/etc/kubernetes/manifests$

```


## APPLICATION LIFECYCLE MANAGEMENT 


### ROLLING UPGRADES AND ROLLBACKS

Revisions/Rollouts helps us keep track of the changes made to a deployment/application during it's lifecycle.  When we create a deployment it triggers a rollout. When we update a deployment it is revised and k8s can help us keep a track of all the revisions.


#### Deployment Strategies

a) <b>RollingUpgrade</b> - : This is the default deployemnt strategy of a k8s deployment. In this, once we upgrade the deployment, the pods associated with a deployement's current replicaSet are deleted one by one and at the same time a new replicaSet is created and new pods are scaled/created. This ensures that the application is UP and running and there is no Downtime as not all Pods are deleted after an update like in ```Recreate``` upgrade strategy. 

b) <b>Recreate</b> - : In this upgrade strategy the pods within the current running ReplicaSet are deleted at once, and then a new replicaSet is created and new pods are brought up. But the drawback is during the time the older ReplicaSet/pods are deleted, the appplication will be <b>DOWN</b>.


### Useful commands to update and create deployments


| Operation | Command |
| --- | --- |
| Create | ```kubectl create -f deployment1.yaml``` |
| Get | ```kubectl get deployment``` |
| Update | ```kubectl apply -f deployment1.yaml``` or ```kubectl set image deploy my-app <container_name>=<new_image>``` |
| Status | ```kubectl rollout status deploy my_app``` and  ```kubectl rollout history deploy my_app``` |
| Rollback to previous revision | ```kubectl rollout undo deploy my_app``` |



### Commands and arguments 

This is how we add a command to a pod definition:

```yaml
apiVersion: v1
kind: Pod 
metadata:
  name: ubuntu-sleeper-3
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
     - "sleep"
     - "1200"
     
     #another ways to add commands
    #command: ["sleep"]
    #args: ["5000"]
    
    #or
    
    #command: ["sleep", "5000"]
      
```



## CONFIGMAPS, SECRETS AND ENVIRONMENT VARIABLES

```ConfigMaps``` aare used to store non-confidential data in ```<key:value>``` pairs. Pods can consume ```ConfigMaps``` as Environment variables, command line arguments, or as configuration files.

https://kubernetes.io/docs/concepts/configuration/configmap/

### Creating a ConfigMap

1) Imperative Command- Using ```kubectl create configmap <name> --from-literal=<key>=<value>``` or ```kubectl create configmap <name> --from-file=<path_to_config_map_file> ```

Example:

```yaml

kubectl create configmap webapp-color-map --from-literal=APP_COLOR=darkblue

```

2) Declarative way - By creating a configMap definiton file and using ```kubectl create -f``` or ```kubectl apply -f```

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config-map
 
 data: #<key:value> pairs go here, cmdline args etc
  APP_COLOR: darkblue

```


Now, once we have the config maps created with the required data, we need to inject them to the Pod.

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
  
spec:
  containers:
  - envFrom:
    - configMapRef:
        name: webapp-config-map

    image: nginx
    

```

Or we can add single eng variables separately from the ConfigMap or add them from a Volume

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
  
spec:
  containers:
  - env:
    - name: APP_COLOR # Define the environment variable
      valueFrom:
        configMapKeyRef:
          name: webapp-config-map. # The ConfigMap this value comes from.
          key: APP_COLOR # The key to fetch.
    

    image: nginx
    
  ```  
  
  
  ```yaml
  
  volumes:
  - name: app-config-volume
    configMap:
      name: app-config
      
 ```
  
  
   
 ### Secrets
    
Secrets are used to store sensitive data like passwords and keys. They are stored in an encoded formate to provide security. We first create a secret and then inject it inside the pod.

https://kubernetes.io/docs/concepts/configuration/secret/

Imperetive command to create a secret:

```yaml

kubectl create secret generic db-secret --from-literal=DB_Host=c3FsMDE= --from-literal=DB_User=root

```

Declarative way of creating a secret:

```yaml
apiVersion: v1

data:
  DB_Host: YzNGc01ERT0=
  DB_User: cm9vdA==
  DB_Password: cGFzc3dvcmQxMjM=
kind: Secret

metadata:
  name: db-secret
  
```


Now, the next step is to inject the secret to the pod, that is done by adding the below :

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
  
spec:
  containers:
  - envFrom:
    - secretRef:
        name: db-secret
        
    image: nginx

```

OR adding a single ENV variable from the secret

```yaml


spec:
  containers:
  - env:
    - name: DB_Host
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: DB_host
        
```

Or pass a secret as files in a Volume:

```yaml

volumes:
 - name: db-secret-volume
   secret:
    secretName: db-secret

```


### Multi Container Pods

Multi container pods share the same lifecycle i.e they are created together and destroyed together. They share the same network space(Connect with each other over localhost) and access the same storage volumes. 

We can make those containers communicate with each other using a Shared Volume(https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)


Spec for multicontainer pods:

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: yellow
  
spec:  
  containers:
  - image: busybox
    name: lemon
    command: ["sleep","1000" ]

  - image: redis
    name: gold
    
```

### initContainers

These are processes/tasks which will run only once during the startup of the pod or when the pod is in Creation state.  These are the processes which which run initially when a pod is created, it is only after they successfully run, other containers are started and run. 

When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts. 

You can configure multiple such initContainers as well, like how we did for multi-pod containers. In that case each init container is run one at a time in sequential order.

InitContainers are terminated once they complete their task. 

<b>NOTE : If any initContainers fail to complete, k8s restarts the pod repeatedly until the INIT container executes successfully. </b>


```yaml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']


```




## Cluster Maintenance


### Working with ```etcdctl```


Backing up ```etcd``` cluster

Using ```etcdctl```:

Working with ETCDCTL


etcdctl is a command line client for etcd.



In all our Kubernetes Hands-on labs, the ETCD key-value database is deployed as a static pod on the master. The version used is v3.

To make use of etcdctl for tasks such as back up and restore, make sure that you set the ```ETCDCTL_API``` to 3.



You can do this by exporting the variable ```ETCDCTL_API```  

```shell
export ETCDCTL_API=3

```

To see all the options for a specific sub-command, make use of the -h or --help flag.



For example, if you want to take a snapshot of etcd, use:

```shell 
etcdctl snapshot save -h
``` 
And keep a note of the mandatory global options.



Since our ETCD database is TLS-Enabled, the following options are mandatory:

1) Verify certificates of TLS-enabled secure servers using this CA bundle

```shell
--cacert
``` 

2)Identify secure client using this TLS certificate file      

```shell 
--cert

```                                             

3) This is the default as ETCD is running on master node and exposed on localhost 2379.
     
```shell 
--endpoints=[127.0.0.1:2379]
```

4) Identify secure client using this TLS key file 

```shell 
--key

```                                                   

5) Similarly use the help option for snapshot restore to see all available options for restoring the backup.

```shell 
etcdctl snapshot restore -h
```

Example, in ```kubeadm``` deployment, ```etcd``` server is deployed as a Static pod on the Master Node.

### Taking a snapshot of the ```etcd``` cluster using ```etcdctl``` :

1) Get the details of the ```etcd``` pod and make a note of the above mentioned parameters, because we need them to run the ```etcdctl``` command:

```shell

>kubectl describe pod etcd-controlplane -n kube-system

Name:                 etcd-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 controlplane/10.64.172.9
Start Time:           Wed, 30 Nov 2022 19:15:39 +0000
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.64.172.9:2379
                      kubernetes.io/config.hash: f558c848c699881551a31da20f182be1
                      kubernetes.io/config.mirror: f558c848c699881551a31da20f182be1
                      kubernetes.io/config.seen: 2022-11-30T19:15:38.717633568Z
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   10.64.172.9
IPs:
  IP:           10.64.172.9
Controlled By:  Node/controlplane
Containers:
  etcd:
    Container ID:  containerd://d34944f57bfddb012d0a3b4ed0323ba6235393062e06310ab4dbe9c5532dfcdd
    Image:         k8s.gcr.io/etcd:3.5.3-0
    Image ID:      k8s.gcr.io/etcd@sha256:13f53ed1d91e2e11aac476ee9a0269fdda6cc4874eba903efd40daf50c55eee5
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://10.64.172.9:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --experimental-initial-corrupt-check=true
      --initial-advertise-peer-urls=https://10.64.172.9:2380
      --initial-cluster=controlplane=https://10.64.172.9:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://10.64.172.9:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://10.64.172.9:2380
      --name=controlplane
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    State:          Running
      Started:      Wed, 30 Nov 2022 19:15:41 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
      memory:     100Mi
    Liveness:     http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
      /var/lib/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:            <none>



```
In the above output we need to make a note of the following - 

```
--cert-file=/etc/kubernetes/pki/etcd/server.crt, --key-file=/etc/kubernetes/pki/etcd/server.key , --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt , 
--listen-client-urls=https://127.0.0.1:2379

```

We will pass the above while running the ```etcdctl``` command  for taking a snapshot:

2) Now we will take the snapshot using the below command. First set the env variable to use the ```etcdctl``` cmds:

```shell

> export ETCDCTL_API=3

> etcdctl snapshot save /opt/snapshot-pre-boot.db \
  > --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  > --endpoints=https://127.0.0.1:2379 \
  > --cert=/etc/kubernetes/pki/etcd/server.crt \
  > --key=/etc/kubernetes/pki/etcd/server.key 
Snapshot saved at /opt/snapshot-pre-boot.db

```

In the above the backup of the etcd cluster is stored in location ```/opt/snapshot-pre-boot.db ```. Use command ```etcdctl snapshot -h``` for more details on using the command. 

Get the status of the snapshot by using the command:

```shell
> etcdctl snapshot status /opt/snapshot-pre-boot.db
1f793c66, 2392, 828, 2.0 MB

```


### Restoring ```etcd``` cluster


1) Restore the snapshot and add a new data directory location to store the old healthy k8s cluster state:

```shell
etcdctl snapshot restore /opt/snapshot-pre-boot.db --data-dir /var/lib/etcd-from-backup 

2022-11-30 20:13:34.201609 I | mvcc: restore compact to 1939
2022-11-30 20:13:34.210061 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32

```

2) Now simply update the ```etcd.yaml``` manifest under ```/etc/kubernetes/manifests``` because that's where static pod's configuration/spec is stored in ```kubeadm``` cluster deployements. 

And update the ```etcd``` pod's spec to point to a new data directory ```/var/lib/etcd-from-backup ```.

Now wait for a few minutes for the ```etcd``` pod to be created. And the k8s cluster state is preserved.


### TLS Certificates in K8s

#### Creating a Certificate Signing Request 



```yaml

openssl genrsa -out akshay.key 2048
openssl req -new -key akshay.key -out akshay.csr

# converted the key to base64 and add it to requests section of the below spec


cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
  - system:authenticated
  request: $(cat akshay.csr | base64 -w 0)
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF

```

Now we can approve or deny the request for the users using the below commands:

```yaml

kubectl certificate approve <csr_object_name>

```

Deny the request:

```yaml

kubectl certificate deny <csr_object_name>

```
