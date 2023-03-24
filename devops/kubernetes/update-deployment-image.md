# Deployment Rollout

## Task

> We have an application running on Kubernetes cluster using nginx web server. The Nautilus application development team has pushed some of the latest changes and those changes need be deployed. The Nautilus DevOps team has created an image `nginx:1.19` with the latest changes.<br><br>Perform a rolling update for this application and incorporate nginx:1.19 image. The deployment name is nginx-deployment<br>Make sure all pods are up and running after the update.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Rolling updates - https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment

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

```bash
# Check the deployments
k get deploy
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           7m51s
```

```bash
# Check the deployments
k describe deploy
```

```
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Fri, 24 Mar 2023 06:26:56 +0000
Labels:                 app=nginx-app
                        type=front-end
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx-app
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-app
  Containers:
   nginx-container:
    Image:        nginx:1.16
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
NewReplicaSet:   nginx-deployment-74fb588559 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  8m8s  deployment-controller  Scaled up replica set nginx-deployment-74fb588559 to 3
```

```bash
# Update deployment image
k set image deployment/nginx-deployment nginx-container=nginx:1.19
```

```
deployment.apps/nginx-deployment image updated
```

```bash
# Check image
k describe po | grep -i image:
```

```
    Image:          nginx:1.19
    Image:          nginx:1.19
    Image:          nginx:1.19
```

We are done as the default rollout strategy is rolling update.
