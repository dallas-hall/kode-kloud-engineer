# Deploy Guestbook Application

## Task

> The Nautilus Application development team has finished development of one of the applications and it is ready for deployment. It is a guestbook application that will be used to manage entries for guests/visitors. As per discussion with the DevOps team, they have finalized the infrastructure that will be deployed on Kubernetes cluster. Below you can find more details about it.<br><br>BACK-END TIER<br>Create a deployment named `redis-master` for Redis master.<br>a.) Replicas count should be 1.<br>b.) Container name should be `master-redis-devops` and it should use image `redis`.<br>c.) Request resources as CPU should be `100m` and Memory should be `100Mi`.<br>d.) Container port should be redis default port i.e `6379`.<br>Create a service named `redis-master` for Redis master. `port` and `targetPort` should be Redis default port i.e `6379`.<br>Create another deployment named `redis-slave` for Redis slave.<br>a.) Replicas count should be 2.<br>b.) Container name should be `slave-redis-devops` and it should use `gcr.io/google_samples/gb-redisslave:v3` image.<br>c.) Requests resources as CPU should be `100m` and Memory should be `100Mi`.<br>d.) Define an environment variable named `GET_HOSTS_FROM` and its value should be `dns`.<br>e.) Container `port` should be Redis default port i.e `6379`.<br>Create another service named `redis-slave`. It should use Redis default port i.e `6379`.<br><br>FRONT END TIER<br>Create a deployment named `frontend`.<br>a.) Replicas count should be 3.<br>b.) Container name should be `php-redis-devops` and it should use `gcr.io/google-samples/gb-frontend:v4` image.<br>c.) Request resources as CPU should be `100m` and Memory should be `100Mi`.<br>d.) Define an environment variable named as `GET_HOSTS_FROM` and its value should be `dns`.<br>e.) Container port should be `80`.<br>Create a service named `frontend`. Its type should be `NodePort`, `port` should be `80` and its `nodePort` should be `30009`.<br>Finally, you can check the guestbook app by clicking on + button in the top left corner and Select port to view on Host 1 then enter your nodePort.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Resources
  * https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
* Environment Variables
  * https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/

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
# Create the first deployment.
cat > redis-master.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis-master
  name: redis-master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-master
  template:
    metadata:
      labels:
        app: redis-master
    spec:
      containers:
      - image: redis
        name: master-redis-devops
        resources:
          requests:
            memory: "100Mi"
            cpu: "100m"
        ports:
        - containerPort: 6379
```

Close the file with control + d i.e. `^D`

```bash
# Create the first deployment.
k apply -f redis-master.yaml
```

```
deployment.apps/redis-master created
```

```bash
# Create the first service.
k expose deployment redis-master --port=6379 --target-port=6379 --name=redis-master
```

```
service/redis-master exposed
```

```bash
# Create the second deployment.
cat > redis-slave.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis-slave
  name: redis-slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis-slave
  template:
    metadata:
      labels:
        app: redis-slave
    spec:
      containers:
      - image: gcr.io/google_samples/gb-redisslave:v3
        name: slave-redis-devops
        resources:
          requests:
            memory: "100Mi"
            cpu: "100m"
        ports:
        - containerPort: 6379
        env:
        - name: GET_HOSTS_FROM
          value: "dns"
```

Close the file with control + d i.e. `^D`

```bash
# Create the second deployment.
k apply -f redis-slave.yaml
```

```
deployment.apps/redis-slave created
```

```bash
# Create the second service.
k expose deployment redis-slave --port=6379 --target-port=6379 --name=redis-slave
```

```
service/redis-slave exposed
```

```bash
# Create the third deployment.
cat > frontend.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: frontend
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  strategy: {}
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - image: gcr.io/google-samples/gb-frontend:v4
        name: php-redis-devops
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "100Mi"
            cpu: "100m"
        env:
        - name: GET_HOSTS_FROM
          value: "dns"
```

Close the file with control + d i.e. `^D`

```bash
# Create the third deployment.
k apply -f frontend.yaml
```

```
deployment.apps/frontend created
```

```bash
# Create the third service.
cat > frontend-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: frontend
  name: frontend
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30009
  selector:
    app: frontend
  type: NodePort
```

```bash
# Create the third service.
k apply -f frontend-svc.yaml
```

```
service/frontend exposed
```

Testing in the UI worked. We are done.
