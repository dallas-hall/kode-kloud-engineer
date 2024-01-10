# Python Deployment Troubleshooting

## Task

> One of the DevOps engineers was trying to deploy a python app on Kubernetes cluster. Unfortunately, due to some mis-configuration, the application is not coming up. Please take a look into it and fix the issues. Application should be accessible on the specified nodePort.
>
> * The deployment name is `python-deployment-datacenter`, its using `poroko/flask-demo-appimage`. The deployment and service of this app is already deployed.
> * nodePort should be `32345` and targetPort should be python flask app's default port.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Python Flask
  * https://www.geeksforgeeks.org/how-to-change-port-in-flask-app/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   54m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox. Click the App button gives an nginx 502 bad gate.

```bash
# Check the Deployment, the Pod wasn't running.
k get deploy python-deployment-datacenter

# Check the verbose output.
k describe deploy python-deployment-datacenter
```

```
Name:                   python-deployment-datacenter
Namespace:              default
CreationTimestamp:      Wed, 10 Jan 2024 04:54:40 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=python_app
Replicas:               1 desired | 1 updated | 1 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=python_app
  Containers:
   python-container-datacenter:
    Image:        poroko/flask-app-demo
    Port:         5000/TCP
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
NewReplicaSet:   python-deployment-datacenter-6fdb496d59 (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m26s  deployment-controller  Scaled up replica set python-deployment-datacenter-6fdb496d59 to 1
```

The image is incorrect but the port is correct.

```bash
# Update the image
k set image deployments python-deployment-datacenter python-container-datacenter=poroko/flask-demo-app

# Check the Deployment, the Pod was running.
k get deploy python-deployment-datacenter
```

We can connect fine from Thor jumpbox. Click the App button gives an nginx 502 bad gate.

```bash
# Check the NodePort.
k describe svc python-service-datacenter
```

```
Name:                     python-service-datacenter
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=python_app
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.58.72
IPs:                      10.96.58.72
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32345/TCP
Endpoints:                10.244.0.6:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

The TargetPort is wrong.

```bash
# Fix the NodePort.
k edit svc python-service-datacenter
```

Change 8080 to 5000.

Pressing the App button shows the connection was successful. We are done.
