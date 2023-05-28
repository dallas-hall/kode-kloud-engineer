# ReplicationController

## Task

> The Nautilus DevOps team wants to create a ReplicationController to deploy several pods. They are going to deploy applications on these pods, these applications need highly available infrastructure. Below you can find exact details, create the ReplicationController accordingly.<br><br>Create a ReplicationController using nginx image, preferably with latest tag, and name it as `nginx-replicationcontroller`.<br>Labels `app` should be `nginx_app`, and labels `type` should be `front-end`. The container should be named as `nginx-container` and also make sure replica counts are 3.<br>All pods should be running state after deployment.


## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* ReplicationControllers.\
  * https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES                  AGE    VERSION                          INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane,master   113m   v1.20.5-rc.0.18+c4af4684437b37   172.17.0.2    <none>        Ubuntu 20.10   5.4.0-1093-gcp   containerd://1.5.0-beta.0-69-gb3f240206
```

We can connect fine from Thor jumpbox.

```bash
# Create the ReplicationController file
cat > rc.yml
```

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-replicationcontroller
spec:
  replicas: 3
  selector:
    app: nginx_app
    type: front-end
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
        ports:
        - containerPort: 80
```

Close the file `^D`


```bash
# Create the RC object
k apply -f rc.yaml
```

```
replicationcontroller/nginx-replicationcontroller created
```

```bash
# View the RC object
k get replicationcontrollers
```

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-replicationcontroller   3         3         3       16s
```

We are done.