# Create A ReplicaSet

## Task

> The Nautilus DevOps team is going to deploy some applications on kubernetes cluster as they are planning to migrate some of their existing applications there. Recently one of the team members has been assigned a task to write a template as per details mentioned below:<br><br>Create a ReplicaSet using httpd image with latest tag only and remember to mention tag i.e `httpd:latest` and name it as `httpd-replicaset`<br>Labels `app` should be `httpd_app`, labels `type` should be `front-end`.<br>The container should be named as `httpd-container`; also make sure replicas counts are `4`.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* ReplicaSet
  * https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE     VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   9m48s   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Create ReplicaSet YAML
cat > rs.yaml
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: httpd-replicaset
  labels:
    app: httpd_app
    type: front-end
spec:
  replicas: 4
  selector:
    matchLabels:
      app: httpd_app
  template:
    metadata:
      labels:
        app: httpd_app
    spec:
      containers:
      - name: httpd-container
        image: httpd:latest
        ports:
        - containerPort: 80
```

```bash
# Create ReplicaSet YAML
k apply -f rs.yaml
```

```
replicaset.apps/httpd-replicaset created
```

```bash
# External expose the ReplicaSet.
k expose rs httpd-replicaset --type=NodePort
```

```
service/httpd-replicaset exposed
```

```bash
# Get NodePort.
k get svc
```

```
NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
httpd-replicaset   NodePort    10.96.153.32   <none>        80:30790/TCP   9s
kubernetes         ClusterIP   10.96.0.1      <none>        443/TCP        20m
```

```bash
# Test connectivity.
curl kodekloud-control-plane:30790
```

```html
<html><body><h1>It works!</h1></body></html>
```

We are done.
