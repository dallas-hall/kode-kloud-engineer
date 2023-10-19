# Deployment Rollbacks

## Task

> There is a production deployment planned for next week. The Nautilus DevOps team wants to test the deployment update and rollback on Dev environment first so that they can identify the risks in advance. Below you can find more details about the plan they want to execute.
>
> * Create a namespace `devops`. Create a deployment called `httpd-deploy` under this new namespace, It should have one container called `httpd`, use `httpd:2.4.28` image and 2 replicas. The deployment should use RollingUpdate strategy with `maxSurge=1`, and `maxUnavailable=2`. Also create a NodePort type service named `httpd-service` and expose the deployment on `nodePort: 30008`.
> * Now upgrade the deployment to version `httpd:2.4.43` using a rolling update.
> * Finally, once all pods are updated undo the recent update and roll back to the previous/original version.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Rolling updates.
  * https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE     VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   8m23s   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Create the namespace
k create ns devops

# Create and edit the deployment YAML
cat > deploy.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: httpd-deploy
  name: httpd-deploy
  namespace: devops
spec:
  replicas: 2
  selector:
    matchLabels:
      app: httpd-deploy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: httpd-deploy
    spec:
      containers:
      - image: httpd:2.4.28
        name: httpd
        resources: {}
        ports:
        - containerPort: 80
status: {}
```

Close the file with control + d i.e. `^D`

```bash
# Deploy the deployment
k apply -f deploy.yaml

# Create and edit the service YAML
cat > svc.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: httpd-deploy
  name: httpd-service
  namespace: devops
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30008
  selector:
    app: httpd-deploy
  type: NodePort
status:
  loadBalancer: {}
```

Close the file with control + d i.e. `^D`

```bash
# Deploy the service
k apply -f svc.yaml

# Update the image
k -n devops set image deployment/httpd-deploy httpd=httpd:2.4.43

# Check the image version
k -n devops describe po | grep Image:
```

```
    Image:          httpd:2.4.43
    Image:          httpd:2.4.43
```

```bash
# Check connectivity.
curl kodekloud-control-plane:30008
```

```html
<html><body><h1>It works!</h1></body></html>
```

```bash
# Rollback the deployment
k -n devops rollout undo deployment httpd-deploy

# Check the image version
k -n devops describe po | grep Image:
```

```
    Image:          httpd:2.4.28
    Image:          httpd:2.4.28
```

```bash
# Check connectivity.
curl kodekloud-control-plane:30008
```

```html
<html><body><h1>It works!</h1></body></html>
```

We are done.
