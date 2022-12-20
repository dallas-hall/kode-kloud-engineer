# nginx Deployment

## Task

> Some of the Nautilus team developers are developing a static website and they want to deploy it on Kubernetes cluster. They want it to be highly available and scalable. Therefore, based on the requirements, the DevOps team has decided to create a deployment for it with multiple replicas. Below you can find more details about it:<br><br>Create a deployment using `nginx` image with `latest` tag only and remember to mention the tag i.e nginx:latest. Name it as `nginx-deployment`. The container should be named as `nginx-container`, also make sure replica counts are 3.<br>Create a NodePort type service named `nginx-service`. The nodePort should be `30011`.<br>Note: The `kubectl` utility on jump_host has been configured to work with the kubernetes cluster.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Deployment - https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
* NodePort - https://kubernetes.io/docs/concepts/services-networking/service/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES                  AGE   VERSION                          INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane,master   91m   v1.20.5-rc.0.18+c4af4684437b37   172.17.0.2    <none>        Ubuntu 20.10   5.4.0-1093-gcp   containerd://1.5.0-beta.0-69-gb3f240206
```

```bash
# Create and edit the Deployment template.
k create deployment nginx-deployment --image=nginx:latest --port=80 --replicas=3 --dry-run -o yaml > deploy.yaml
vi deploy.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:latest
        name: nginx-container
        ports:
        - containerPort: 80
        - containerPort: 443
```

```bash
# Create the deployment
k apply -f deploy.yaml
```

```
deployment.apps/nginx-deployment created
```

```bash
# Create and edit the NodePort template for the Deployment.
k expose deployment nginx-deployment --port=80 --type=NodePort --name=nginx-service --dry-run -o yaml > nodeport.yaml
vi nodeport.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-deployment
  name: nginx-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30011
    name: http
  - port: 443
    protocol: TCP
    targetPort: 80
    nodePort: 30443
    name: https
  selector:
    app: nginx-deployment
  type: NodePort
```

```bash
# Create the NodePort
k apply -f nodeport.yaml
```

```
service/nginx-service created
```

Tested the connection by clicking the App button and saw the 'Welcome to nginx!' page, we are done.
