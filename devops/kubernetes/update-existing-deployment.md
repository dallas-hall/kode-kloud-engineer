# Update Existing Deployment

## Task

> There is an application deployed on Kubernetes cluster. Recently, the Nautilus application development team developed a new version of the application that needs to be deployed now. As per new updates some new changes need to be made in this existing setup. So update the deployment and service as per details mentioned below:
>
> We already have a deployment named `nginx-deployment` and service named `nginx-service`. Some changes need to be made in this deployment and service, make sure not to delete the deployment and service.
>
> 1.) Change the service nodeport from 30008 to 32165
>
> 2.) Change the replicas count from 1 to 5
>
> 3.) Change the image from `nginx:1.19` to `nginx:latest`

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* None.

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE     VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   9m30s   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Check the Deployment.
k get deploy
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           3m8s
```

```bash
# Check the Deployment image.
k describe deploy
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Wed, 06 Sep 2023 08:08:01 +0000
Labels:                 app=nginx-app
                        type=front-end
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx-app
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-app
  Containers:
   nginx-container:
    Image:        nginx:1.19
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-dc49f85cc (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  7m    deployment-controller  Scaled up replica set nginx-deployment-dc49f85cc to 1
```

</details>

```bash
# Check the NodePort Service.
k get svc
```

```
NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1     <none>        443/TCP        10m
nginx-service   NodePort    10.96.2.153   <none>        80:30008/TCP   3m35s
```

```bash
# Test the NodePort Service
curl kodekloud-control-plane:30008
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
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

```bash
# Update the Deployment image.
k set image deployment/nginx-deployment nginx-container=nginx:latest
```

```
deployment.apps/nginx-deployment image updated
```

```bash
# Scale the Deployment.
k scale deployment nginx-deployment --replicas=5
```

```
deployment.apps/nginx-deployment scaled
```

```bash
# Check the Deployment.
k get deploy
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5/5     5            5           8m50s
```

```bash
# Check the Deployment image.
k describe deploy | grep -i image:
```

```
    Image:        nginx:latest
```

```bash
# Update the NodePort's nodePort
k edit svc nginx-service
```

Update the line `  - nodePort: 30008` to `  - nodePort: 32165` and save with `:x`.

```
service/nginx-service edited
```

```bash
# Check the NodePort Service.
k get svc
```

```
NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1     <none>        443/TCP        17m
nginx-service   NodePort    10.96.2.153   <none>        80:32165/TCP   10m
```

```bash
# Test the NodePort Service
curl kodekloud-control-plane:32165
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

