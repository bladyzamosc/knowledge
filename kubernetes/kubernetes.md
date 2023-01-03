## Kubernetes 

### 1. Glossary and tools

- kubectl - The tool for helping manage the Kubernetes cluster and application.
- kubelet - kubernetes agent

### 2. Kubernetes

Kubernetes is an orchestrator of cloud-native microservices applications.

### 3. Cloud native

- scale on demand 
- self-heal
- suppot zero-downtime tolling updates 
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

### 8. Service

```
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
