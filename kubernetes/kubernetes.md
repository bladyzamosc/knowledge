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

### 6. Pod

- wrapper around the container 
- container runs inside the pod

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

 - service

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
