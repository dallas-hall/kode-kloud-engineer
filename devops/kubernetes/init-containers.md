# Create A Deployment With An initContainer

## Task

> There are some applications that need to be deployed on Kubernetes cluster and these apps have some pre-requisites where some configurations need to be changed before deploying the app container. Some of these changes cannot be made inside the images so the DevOps team has come up with a solution to use init containers to perform these tasks during deployment. Below is a sample scenario that the team is going to test first.
>
> * Create a Deployment named as `ic-deploy-xfusion`.
> Configure spec as replicas should be `1`, labels `app` should be `ic-xfusion`, template's metadata lables `app` should be the same `ic-xfusion`.
> * The `initContainers` should be named as `ic-msg-xfusion`, use image `debian`, preferably with `latest` tag and use command `'/bin/bash', '-c', 'echo Init Done - Welcome to xFusionCorp Industries > /ic/blog'`. The volume mount should be named as `ic-volume-xfusion` and mount path should be `/ic`.
> * Main container should be named as `ic-main-xfusion`, use image `debian`, preferably with `latest` tag and use command `'/bin/bash', '-c', 'while true; do cat /ic/blog; sleep 5; done'`. The volume mount should be named as `ic-volume-xfusion` and mount path should be `/ic`.
> * Volume to be named as `ic-volume-xfusion` and it should be an `emptyDir` type.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Init Containers
  * https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
* Volumes
  * https://kubernetes.io/docs/concepts/storage/volumes/#emptydir

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   85m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Create Deployment YAML
cat > d.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: ic-xfusion
  name: ic-deploy-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-xfusion
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: ic-xfusion
    spec:
      initContainers:
      - name: ic-msg-xfusion
        image: debian:latest
        command: ['/bin/bash', '-c', 'echo Init Done - Welcome to xFusionCorp Industries > /ic/blog']
        volumeMounts:
        - mountPath: /ic
          name: ic-volume-xfusion
      containers:
      - image: debian:latest
        name: ic-main-xfusion
        command: ['/bin/bash', '-c', 'while true; do cat /ic/blog; sleep 5; done']
        volumeMounts:
        - mountPath: /ic
          name: ic-volume-xfusion
      volumes:
      - name: ic-volume-xfusion
        emptyDir:
          sizeLimit: 128Mi

```
Close the file with control + d i.e. `^D`


```bash
# Create the Deployment
k apply -f  d.yaml

# Get the Pod name.
k get po

# Check the logs of the main container.
k logs ic-deploy-xfusion-69699c6576-x4vg4 -c ic-main-xfusion
```

```
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
```

We are done.
