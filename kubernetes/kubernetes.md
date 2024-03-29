## Kubernetes 

### 1. Glossary and tools

- kubectl - The tool for helping manage the Kubernetes cluster and application.
- kubelet - kubernetes agent

### 2. Kubernetes

Kubernetes is an orchestrator of cloud-native microservices applications.

- containers cannot be deployed directly, because they not provide fault tolerance
- containers cannot be deployed with optimum spot in the cluster

### 3. Cloud native

- scale on demand 
- self-heal
- support zero-downtime tolling updates 
- run anywhere that has Kubernetes - AWS, Azure, Linode, your on-premises datacenter, or even your Raspberry Pi cluster at home.

### 4. Masters and nodes 

- masters - host control pane
- nodes run user applications

### 5. Commands 

- kubectl get nodes
- export KUBECONFIG=/usecode/config
- kubectl apply -f pod.yml
- kubectl get pods
- kubectl port-forward --address 0.0.0.0 first-pod 8080:8080
- kubectl describe pod first-pod
- kubectl logs --follow nginx // streaming logs
- kubectl exec nginx -- ls // executing commands
- kubectl exec -it nginx -- bash // start bash
- kubectl delete pod nginx // killing pods
- kubectl delete -f nginx.yaml // killing pods 
- kubectl rollout undo deployment hello k8s
- k3d cluster create mycluster

### 6. Pod

- wrapper around the container 
- container runs inside the pod
- each pod has 1 and more containers
- Pods are atomic units for scheduling - 10 replicas of our application, we would create 10 pods instead of creating one pod with 10 containers
- basic building block and the atomic unit of scheduling in Kubernetes
- when a pod dies, either because we manually kill it or because something unexpected happens, Kubernetes will not automatically reschedule it.

**Manifest**
```
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  labels:
    project: example-app
spec:
  containers:
    - name: bladyzamosc-app
      image: bladyzamosc/example-app:1.0
      ports:
        - containerPort: 8080
```

### 7. Multicontainer pods 

- pods are very similar to containers,but we can have any containers working inside a pod
- pod is a wrapper for a group of containers

### 8. Services

- provides a stable enpoint for pods
- it is something, which sits in front of pods and takes care of receibing requires and delivering them to all pods that are behind it
- NodePort will open a port on the woeker node to reveive requests (similary to port-forward)

```
apiVersion: v1
kind: Service
metadata:
  name: cloud-lb
spec:
  type: NodePort
  ports: // wea open the port 30001 and send it to 4567
  - port: 4567
    nodePort: 30001 
  selector:
    project: bladyzamosc-app
    

apiVersion: v1
kind: Service
metadata:
  name: cloud-lb
spec:
  type: LoadBalancer
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    project: bladyzamosc-app
```

**Service types**
- ClusterIP - default, we only need to give other applications that are running inside our cluster access to our pods. 

```
pod A -> Service -> pod B 1
                 -> pod B 2
                 -> pod B 3
```

- NodePort - extension for ClusterIP, but additionally allowing application that in our cluster to talk to each of each other, it will also allow us to expose our application to the outside world. It works by opening a port on all the worker nodes we have in our cluster, and then redirecting requests received on that port to the correct location, even if the pod we are trying to reach is physically running on a different node
- LoadBalancer - extension for NodePort, provision a Load Balancer on the cloud provider we are running, for example it will createt ELB if cloud provider is AWS
- External name

```
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  type: ExternalName
  externalName: baza.bladyzamosc.com
```

**DNS**

- works out of the box
- any type of service
- its preffered for service discovery mechanism

### 9. Deployments

- We will almost always want to manage pods with another Kubernetes resource called Deployment
- making sure our pods are rescheduled when they die
- We can use deployments to scale our applications by increasing or decreasing the number of replicas we have running. So far, we have only run one replica of our application.
- rollout from v1 to v2
- rollback bad releases

```
apiVersion: apka/v1
kind: Deployment
metadata:
  name: examplek8s
spec:
  replicas: 1
  selector:
    matchLabels:
      app: examplek8s
  template:
    metadata:
      labels:
        app: examplek8s
    spec:
      containers:
      - image: abc/examplek8s:v1
        name: examplek8s-container
```

- replicas - how many copies we want to have 
- selector - telling Kubernetes that this deployment is managing all the pods that have a label called app = examplek8s. This links deployments to pods
- template - definition of all pods we want to run.
- kubectl get deployments


### 10. Deployments - killing pods  
```
Kubectl apply -f deployment.yaml
kubectl get pods

# NAME                        READY   STATUS    RESTARTS
# hellok8s-1111111-1111  1/1     Running   0 

kubectl delete pod hellok8s-6678f66cb8-42jtr
kubectl get pods

# NAME                        READY   STATUS    RESTARTS  
# hellok8s-1111111-2222   1/1     Running   0   
```

- killing pod is substituted by another pod

### 11. Scaling out

- spec.replicas: number

### 12. Rollout

- maxSurge - how many replicas we can exeeding our esired replica count
- maxUnavailable - how many pods we can have below this count

```
...
spec:
    strategy:
      type: Recreate
    replicas: 3
    strategy:
        rollingUpdate:
            maxSurge: 1
            maxUnavailable: 1
...
```

- rollingUpdate - is default strategy - during some period it will be 2 version of the application 
- recreate - all pods will be terminated and pods with a new version will be created
- livenessProbe - 
- readinessProbe - 


### 13. Ingress

- for exposing http routes 
- ingress sits before services and based on rules the request will be sent
- ingress controller - azure ingress controller

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bladyzamosc-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /one
            pathType: Prefix
            backend:
              service:
                name: service-one
                port:
                  number: 1234
          - path: /two
            pathType: Prefix
            backend:
              service:
                name: service-two
                port:
                  number: 4567
```

### 14. Config map

- ConfigMaps can be used for configuration instead of defining in the docker image
- one-by-one or use envFrom to get all the variables from a ConfigMap
- prefix property for avoiding variables conflicts.
- ConfigMaps can also be mounted as files in the container

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: bladyzamosc-config
data:
  MESSAGE: "It works with a ConfigMap!"
  
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bladyzamosc
spec:
  replicas: 5
  selector:
    matchLabels:
      app: bladyzamosc
  template:
    metadata:
      labels:
        app: bladyzamosc
    spec:
      containers:
      - image: bladyzamosc/example:v2
        name: bladyzamosc-container
        env:
          - name: MESSAGE
            valueFrom:
              configMapKeyRef:
                name: bladyzamosc-config
                key: MESSAGE
```

### 15. Secrets

- similar to configMaps
- only distributed to nodes that are running a pod that needs them
- never written to physical storage
- in the master they are encrypted 
- reading a SECRET we are getting base64 encoded 
- configMapKeyRef in ConfigMap and here secretKeyRef, secretRef
- stringData - if we annoyed of base64
- mounting similar like in ConfigMaps


### 16. Running jobs

- Job is a resource that will create pods, will manage them and exit successfully if this will be required. When all pods will be completed job will be marked as completeted too.
- restart policy - when pod should be restarted - always (default, it will be restarted indepedently on success or failure), onFailure (only if fails), never
- pod will not be deleted after exits and that let to inspect it
- ttlSecondsAfterFinished - after how many seconds job will be automatically deleted
- completions (default = 1) - it will create 4 pods in row one ofter another 
- parallelism - changes behaviour of above letting to run 2 pods at the same time
- backoffLimit (default = 6) - it will keep retrying 10 times, still using an exponential back-of delay each retry 
- activeDeadlineSeconds - how long job is allowed to run, after that job will fail
- schedule + kind: ChronJob - (e.g. "* * * * *" - every minute) - letting define chron job 

```
apiVersion: batch/v1
kind: Job
metadata:
  name: bladyzamosc-job
spec:
  parallelism: 2
  completion: 4
  backoffLimit: 10
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: bladyzamosc
        image: bladyzamosc
        command: ["hello", "Running in a bladyzamosc job"]
```

### 17. Namespaces 

```
apiVersion: v1
kind: Namespace
metadata:
  name: bladyzamosc-namespace
  
  kubectl apply -f namespace.yaml
  kubectl get namespaces
  
  kubectl get pods --all-namespaces
  kubectl get all --all-namespaces
  kubectl apply -f bladyzamosc-pod.yaml -n my-namespace
```

### 18. Resources 

- Containers without resource limits can affect other applications sharing the same worker node
- CPU - compressible resource, which means Kubernetes can just stop giving that specific container CPU time if a container tries to acquire more CPU than it should
- memory - is containers tries to allocate more that it can Kubernetes will kill the process and depending on restart policy do the job

```
apiVersion: v1
kind: Pod
metadata:
  name: bladyzamosc
spec:
  containers:
  - name: bladyzamosc-container
    image: bladyzamosc
    command: ["dd", "if=/dev/zero", "of=/dev/null"]
    resources:
      requests:
        cpu: 500m
        memory: 5Mi
      limits:
        cpu: 500m
        memory: 10Mi
```

- requests - this means 1/2 of single CPU core and 5MB of memory
- limits - limited to 1/2 of CPU core 
- LimitRange - cna be applied to namespace (kubectl apply -f default-limit-range.yaml)

```
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-limit-range
spec:
  limits:
  - default:
      memory: 500Mi
      cpu: 200m
    defaultRequest:
      memory: 100Mi
      cpu: 100m
    type: Container
```

### 19. Config

- clusters 
- users - how we are going to authenticate against cluster
- contexts - combination of cluster and users 

```
apiVersion: v1
kind: Config
current-context: docker-for-desktop

clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://localhost:6443
  name: docker-for-desktop-cluster

users:
- name: docker-for-desktop
  user:
    client-certificate-data: LS0tL...
    client-key-data: LS0tL...

contexts:
- context:
    cluster: docker-for-desktop-cluster
    user: docker-for-desktop
  name: docker-for-desktop
  
  
  kubectl config current-context
  kubectl config use-context production
  kubect get pods --context production
  kubectl --kubeconfig="path/to/config" get pods
```
