# Iron Gallery Deployment

## Task

> There is an iron gallery app that the Nautilus DevOps team was developing. They have recently customized the app and are going to deploy the same on the Kubernetes cluster. Below you can find more details:
>
> * Create a namespace `iron-namespace-datacenter`
> * Create a deployment `iron-gallery-deployment-datacenter` for iron gallery under the same namespace you created.
>   * Labels `run` should be `iron-gallery`.
>   * Replicas count should be 1.
>   * Selector's matchLabels `run` should be `iron-gallery`.
>   * Template labels `run` should be `iron-gallery` under metadata.
>   * The container should be named as `iron-gallery-container-datacenter`, use `kodekloud/irongallery:2.0` image ( use exact image name / tag ).
>   * Resources limits for memory should be `100Mi` and for CPU should be `50m`.
>   * First volumeMount name should be `config`, its mountPath should be `/usr/share/nginx/html/data`.
>   * Second volumeMount name should be `images`, its mountPath should be `/usr/share/nginx/html/uploads`.
>   * First volume name should be `config` and give it `emptyDir` and second volume name should be `images`, also give it `emptyDir`.
> * Create a deployment `iron-db-deployment-datacenter` for iron db under the same namespace.
>   * Labels `db` should be `mariadb`.
>   * Replicas count should be 1.
>   * Selector's matchLabels `db` should be `mariadb`.
>   * Template labels `db` should be `mariadb` under metadata.
>   * The container name should be `iron-db-container-datacenter`, use `kodekloud/irondb:2.0` image ( use exact image name / tag ).
>   * Define environment, set `MYSQL_DATABASE` its value should be `database_blog`, set `MYSQL_ROOT_PASSWORD` and `MYSQL_PASSWORD` value should be with some complex passwords for DB connections, and `MYSQL_USER` value should be any custom user ( except root ).
>   * Volume mount name should be `db` and its mountPath should be `/var/lib/mysql` Volume name should be `db` and give it an `emptyDir`.
>  * Create a service for iron db which should be named `iron-db-service-datacenter` under the same namespace. Configure spec as selector's `db` should be `mariadb`. Protocol should be TCP, `port` and `targetPort` should be `3306` and its type should be `ClusterIP`.
> * Create a service for iron gallery which should be named `iron-gallery-service-datacenter` under the same namespace. Configure spec as selector's `run` should be `iron-gallery`. Protocol should be TCP, `port`and `targetPort` should be `80`, `nodePort`should be `32678` and its type should be `NodePort`.
>
> Notes: 
> * We don't need to make connection b/w database and front-end now, if the installation page is coming up it should be enough for now.
> * The `kubectl` on jump_host has been configured to work with the kubernetes cluster.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Environment Variables.
  * https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/
* K8s emptyDir Volume
  * https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
* NodePort
  * https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
* Resource limits
  * https://kubernetes.io/docs/concepts/policy/limit-range/
  * https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/
  * https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   23m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Create the Namespace
k create ns iron-namespace-datacenter

# Create the first Deployment YAML.
cat > deploy1.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-datacenter
  namespace: iron-namespace-datacenter
  labels:
    run: iron-gallery
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
      - image: kodekloud/irongallery:2.0
        name: iron-gallery-container-datacenter
        resources:
          limits:
            cpu: 50m # 5% of 1 hyperthread.
            memory: 100Mi # Binary, 100 Mebibytes.
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html/data
          name: config
        - mountPath: /usr/share/nginx/html/uploads
          name: images
      volumes:
      - name: config
        emptyDir:
          sizeLimit: 16Mi # Binary, 16 Mebibytes.
      - name: images
        emptyDir:
          sizeLimit: 128Mi # Binary, 128 Mebibytes.

```

Close the file with control + d i.e. `^D`

```bash
# Create the second Deployment YAML.
cat > deploy2.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-datacenter
  namespace: iron-namespace-datacenter
  labels:
    db: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
      - image: kodekloud/irondb:2.0
        name: iron-db-container-datacenter
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "Abc123!@#"
        - name: MYSQL_DATABASE
          value: "database_blog"
        - name: MYSQL_USER
          value: "irongallery_user"
        - name: MYSQL_PASSWORD
          value: "Abc123!@#"
        ports:
        - containerPort: 3306
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: db
      volumes:
      - name: db
        emptyDir:
          sizeLimit: 1Gi # Binary, 1 Gibibyte.

```

Close the file with control + d i.e. `^D`


```bash
# Create the Deployments.
k apply -f deploy1.yaml
k apply -f deploy2.yaml

# Create the NodePort YAML.
cat > np.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: iron-gallery
  name: iron-gallery-service-datacenter
  namespace: iron-namespace-datacenter
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 32678
  selector:
    run: iron-gallery
  type: NodePort
status:
  loadBalancer: {}
```

```bash
# Create the NodePort.
k apply -f np.yaml

# Create the ClusterIP.
k -n iron-namespace-datacenter expose deployment iron-db-deployment-datacenter --port 3306 --name iron-db-service-datacenter
```

Click the App button and view the installation page, which means the NodePort is working.

We are done.
