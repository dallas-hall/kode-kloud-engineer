# Redis Deployment

## Task

> The Nautilus application development team observed some performance issues with one of the application that is deployed in Kubernetes cluster. After looking into number of factors, the team has suggested to use some in-memory caching utility for DB service. After number of discussions, they have decided to use Redis. Initially they would like to deploy Redis on kubernetes cluster for testing and later they will move it to production. Please find below more details about the task:
>
> Create a redis deployment with following parameters:
> * Create a config map called `my-redis-config` having `maxmemory 2mb` in `redis-config`.
> * Name of the deployment should be `redis-deployment`, it should use `redis:alpine` image and container name should be `redis-container` Also make sure it has only 1 replica.
> * The container should request for 1 CPU.
> * Mount 2 volumes:
>   * An Empty directory volume called `data` at path `/redis-master-data`.
>   * A configmap volume called `redis-config` at path `/redis-master`.
> * The container should expose the port `6379`.
> Finally, `redis-deployment` should be in an up and running state.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* ConfigMap
  * https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/
* Deployments
  * https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
* emptyDir
  * https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
* ConfigMap as Volume.
  * https://kubernetes.io/docs/concepts/storage/volumes/#configmap
* Volumes
  * https://kubernetes.io/docs/concepts/storage/volumes

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   43m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Create the ConfigMap
cat > cm.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
data:
  redis-config: |
    maxmemory 2mb
```

This is equal to the below:

```bash
echo 'maxmemory 2mb' > /tmp/redis-config
k create cm my-redis-config --from-file /tmp/redis-config

# Create the ConfigMap object.
k create -f cm.yaml

# Check the ConfigMap object, looks okay.
k describe cm my-redis-config

# Create Deployment YAML.
cat > d.yml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis-deployment
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: redis-deployment
    spec:
      containers:
      - image: redis:alpine
        name: redis-container
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "1"
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: config
      volumes:
        - name: data
          emptyDir: {}
        - name: config
          configMap:
            name: my-redis-config
            items:
            - key: redis-config
              path: redis.conf
```

```bash
# Create the Deployment
k apply -f d.yml

# Check the Deployment and its Pods, they were both fine.
k get deploy
k get po
```

We are done.
