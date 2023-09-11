# ReplicationController

## Task

> The Nautilus DevOps team wants to create a ReplicationController to deploy several pods. They are going to deploy applications on these pods, these applications need highly available infrastructure. Below you can find exact details, create the ReplicationController accordingly.
>
> Create a ReplicationController using `httpd` image, preferably with `latest` tag, and name it as `httpd-replicationcontroller`.
>
> Labels `app` should be `httpd_app`, and labels `type` should be `front-end`. The container should be named as `httpd-container` and also make sure replica counts are 3.
>
> All pods should be running state after deployment.


## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* ReplicationControllers.
  * https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   14m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
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
  name: httpd-replicationcontroller
spec:
  replicas: 3
  selector:
    app: httpd_app
    type: front-end
  template:
    metadata:
      name: httpd
      labels:
        app: httpd_app
        type: front-end
    spec:
      containers:
      - name: httpd-container
        image: httpd:latest
        ports:
        - containerPort: 80
```

Close the file `^D`


```bash
# Create the RC object
k apply -f rc.yaml
```

```
replicationcontroller/httpd-replicationcontroller created
```

```bash
# View the RC object
k get rc
```

```
NAME                          DESIRED   CURRENT   READY   AGE
httpd-replicationcontroller   3         3         3       12s
```

```bash
# Test the RC.
k expose rc httpd-replicationcontroller --type=NodePort --port=80
```

```
service/httpd-replicationcontroller exposed
```

```bash
# Get the port for testing.
k get svc
```

```
NAME                          TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
httpd-replicationcontroller   NodePort    10.96.3.215   <none>        80:32498/TCP   3s
kubernetes                    ClusterIP   10.96.0.1     <none>        443/TCP        18m
```

```bash
# Run the test.
curl kodekloud-control-plane:32498
```

```html
<html>
  <body>
    <h1>It works!</h1>
  </body>
</html>
```

We are done.
