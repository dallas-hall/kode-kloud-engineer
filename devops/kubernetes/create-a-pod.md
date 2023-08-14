# Create A Pod

## Task

> The Nautilus DevOps team has started practicing some pods and services deployment on Kubernetes platform as they are planning to migrate most of their applications on Kubernetes platform. Recently one of the team members has been assigned a task to create a pod as per details mentioned below:<br><br>Create a pod named `pod-httpd` using `httpd` image with `latest` tag only and remember to mention the tag i.e httpd:latest.<br>Labels `app` should be set to `httpd_app`, also container should be named as `httpd-container`.

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
kodekloud-control-plane   Ready    control-plane   8m36s   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Create Pod YAML
cat > pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: httpd_app
  name: pod-httpd
spec:
  containers:
  - image: httpd:latest
    name: httpd-container
```
Close the file with control + d i.e. `^D`


```bash
# Create the Pod
k apply -f  pod.yaml
```

```
kubecpod/pod-httpd created
```

```bash
# Check the Pod
k get po
```

```
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   1/1     Running   0          40s
```

We are done.
