# nginx Deployment

## Task

> The Nautilus DevOps team has started practicing some pods, and services deployment on Kubernetes platform, as they are planning to migrate most of their applications on Kubernetes. Recently one of the team members has been assigned a task to create a deploymnt as per details mentioned below:<br><br>Create a deployment named `nginx` to deploy the application `nginx` using the image `nginx:latest` (remember to mention the tag as well)

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

None.

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   44m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Create the Deployment
k create deployment nginx --image=nginx:latest --port=80
```

```
deployment.apps/nginx created
```

```bash
# Check it
k describe deploy
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Name:                   nginx
Namespace:              default
CreationTimestamp:      Sat, 19 Aug 2023 00:07:17 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               1 desired | 1 updated | 1 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   nginx-57d84f57dc (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  5s    deployment-controller  Scaled up replica set nginx-57d84f57dc to 1
```

</details>

```bash
# View the Pod
k get po
```

```
NAME                    READY   STATUS    RESTARTS   AGE
nginx-585449566-q8qzn   1/1     Running   0          97s
```

```bash
# Create a ClusterIP for it
k expose deployment nginx --port=80 --name=nginx-internal
```

```
service/nginx-internal exposed
```


```bash
# Create a NodePort for it
k expose deployment nginx --type=NodePort --name=nginx-external
```

```
service/nginx-external exposed
```

```bash
# View the Services
k get svc
```

```
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP        48m
nginx            ClusterIP   10.96.140.227   <none>        80/TCP         3m25s
nginx-external   NodePort    10.96.22.119    <none>        80:32594/TCP   3s
thor@jump_host ~$ curl kodekloud-control-plane:32594
```

```bash
# Test external connectivity
curl kodekloud-control-plane:32594
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

</details>

We are done.
