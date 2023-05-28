# Deployment Rollbacks

## Task

> There is a production deployment planned for next week. The Nautilus DevOps team wants to test the deployment update and rollback on Dev environment first so that they can identify the risks in advance. Below you can find more details about the plan they want to execute.<br><br>Create a namespace `devops`. Create a deployment called `httpd-deploy` under this new namespace, It should have one container called `httpd`, use `httpd:2.4.25` image and 2 replicas. The deployment should use RollingUpdate strategy with `maxSurge=1`, and `maxUnavailable=2`. Also create a NodePort type service named `httpd-service` and expose the deployment on `nodePort: 30008`.<br>Now upgrade the deployment to version httpd:2.4.43 using a rolling update.<br>Finally, once all pods are updated undo the recent update and roll back to the previous/original version.

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
NAME                      STATUS   ROLES                  AGE    VERSION                          INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane,master   145m   v1.20.5-rc.0.18+c4af4684437b37   172.17.0.2    <none>        Ubuntu 20.10   5.4.0-1092-gcp   containerd://1.5.0-beta.0-69-gb3f240206
```

We can connect fine from Thor jumpbox.

```bash
# Create the namespace
k create ns devops
```

```
namespace/devops created
```

```bash
# Create and edit the deployment YAML
k -n devops create deployment httpd-deploy --image=httpd:2.4.25 --replicas=2 --dry-run -o yaml > deploy.yaml
vi deploy.yaml
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
      - image: httpd:2.4.25
        name: httpd
        resources: {}
        ports:
        - containerPort: 80
status: {}
```

Close the file `:x`

```bash
# Deploy the deployment
k apply -f deploy.yaml
```

```
deployment.apps/httpd-deploy created
```

```bash
# Create and edit the service YAML
k -n devops expose deployment httpd-deploy --type=NodePort --port=80 --name=httpd-service --dry-run -o yaml > svc.yaml
vi svc.yaml
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

Close the file `:x`

```bash
# Deploy the service
k apply -f svc.yaml
```

```
service/httpd-service created
```

```bash
# Update the image
k -n devops set image deployment/httpd-deploy httpd=httpd:2.4.43
```

```
deployment.apps/httpd-deploy image updated
```

```bash
# Check the image version
k -n devops describe po | grep Image:
```

```
    Image:          httpd:2.4.43
    Image:          httpd:2.4.43
```

```bash
# Rollback the deployment
k -n devops rollout undo deployment httpd-deploy
```

```
deployment.apps/httpd-deploy rolled back
```

```bash
# Check the image version
k -n devops describe po | grep Image:
```

```
    Image:          httpd:2.4.25
    Image:          httpd:2.4.25
```
