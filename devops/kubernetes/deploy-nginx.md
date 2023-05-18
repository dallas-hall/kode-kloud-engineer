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
NAME                      STATUS   ROLES                  AGE    VERSION                          INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane,master   145m   v1.20.5-rc.0.18+c4af4684437b37   172.17.0.2    <none>        Ubuntu 20.10   5.4.0-1092-gcp   containerd://1.5.0-beta.0-69-gb3f240206
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
k describe deployments.apps
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Name:                   nginx
Namespace:              default
CreationTimestamp:      Thu, 18 May 2023 09:41:33 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
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
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-585449566 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  39s   deployment-controller  Scaled up replica set nginx-585449566 to 1
```

</details>

```bash
# Create a ClusterIP for it
k expose deployment nginx --port=80
```

```
service/nginx exposed
```

```bash
# Describe the ClusterIP.
k describe svc nginx
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP:                10.96.194.223
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.0.7:80
Session Affinity:  None
Events:            <none>
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

We are done.
