# Deployment Rolling Update

## Task

> We have an application running on Kubernetes cluster using nginx web server. The Nautilus application development team has pushed some of the latest changes and those changes need be deployed. The Nautilus DevOps team has created an image `nginx:1.19` with the latest changes.<br><br>Perform a rolling update for this application and incorporate nginx:1.19 image. The deployment name is nginx-deployment<br>Make sure all pods are up and running after the update.

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
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   10m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Check the deployments
k get deploy
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           87s
```

```bash
# Check the deployments
k describe deploy
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 24 Aug 2023 22:59:24 +0000
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
NewReplicaSet:   nginx-deployment-989f57c54 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  97s   deployment-controller  Scaled up replica set nginx-deployment-989f57c54 to 3
```

</details>

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
