# phpfpm Deployment

## Task

> We need to deploy a Drupal application on Kubernetes cluster. The Nautilus application development team want to setup a fresh Drupal as they will do the installation on their own. Below you can find the requirements, they have shared with us.
> 1) Configure a persistent volume `drupal-mysql-pv` with hostPath = `/drupal-mysql-data` (/drupal-mysql-data directory already exists on the worker Node i.e jump host), `5Gi` of storage and `ReadWriteOnce` access mode.
> 2) Configure one PersistentVolumeClaim named `drupal-mysql-pvc` with storage request of `3Gi` and `ReadWriteOnce` access mode.
> 3) Create a deployment `drupal-mysql` with `1` replica, use `mysql:5.7` image. Mount the claimed PVC at `/var/lib/mysql`.
> 4) Create a deployment `drupal` with 1 replica and use `drupal:8.6` image.
> 4) Create a NodePort type service which should be named as `drupal-service` and nodePort should be `30095`.
> 5) Create a service `drupal-mysql-service` to expose `mysql` deployment on port `3306`.
> 6) Set rest of the configration for deployments, services, secrets etc as per your preferences. At the end you should be able to access the Drupal installation page by clicking on App button.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Environment Variable
  * https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/
* K8s emptyDir Volume
  * https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
  * https://kubernetes.io/docs/concepts/storage/volumes/#configmap
* Config Map
  * https://stackoverflow.com/a/64179010
* NodePort
  * https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   33m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```


```bash
# Create the Persistent Volume file
cat > pv.yml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: drupal-mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/drupal-mysql-data"
```

Close the file with control + d i.e. `^D`

```bash
# Create the Persistent Volume Claim file
cat > pvc.yml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-mysql-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

Close the file with control + d i.e. `^D`

```bash
# Check the binding, it was okay.
k get pv,pvc
```

```
NAME                               CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS   REASON   AGE
persistentvolume/drupal-mysql-pv   5Gi        RWO            Retain           Bound    default/drupal-mysql-pvc   manual                  45s

NAME                                     STATUS   VOLUME            CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/drupal-mysql-pvc   Bound    drupal-mysql-pv   5Gi        RWO            manual         23s
```


```bash
# Create Deployment YAML.
cat > mysql-deploy.yml
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: drupal-mysql
  name: drupal-mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: drupal-mysql
  template:
    metadata:
      labels:
        app: drupal-mysql
    spec:
      containers:
      - image: mysql:5.7
        name: mysql
        ports:
        - containerPort: 3306
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: data
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "Abc123!@#"
        - name: MYSQL_DATABASE
          value: "drupal-db"
        - name: MYSQL_USER
          value: "drupal-user"
        - name: MYSQL_PASSWORD
          value: "Abc123!@#"
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: drupal-mysql-pvc
```

Close the file with control + d i.e. `^D`

```bash
# Create the Deployments.
k apply -f mysql-deploy.yml
k create deployment drupal --image=drupal:8.6 --replicas=1 --port=80

# Wait for the Deployments to be ready.
k get deployments.apps -w
```

```
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
drupal         1/1     1            1           28s
drupal-mysql   1/1     1            1           4m38s
```

```bash
# Create the ClusterIP Service.
k expose deployment drupal-mysql --port=3306 --name=drupal-mysql-service

# Create the NodePort YAML.
cat > np.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: drupal
  name: drupal-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30095
  selector:
    app: drupal
  type: NodePort
```

Close the file with control + d i.e. `^D`

```bash
# Create the NodePort Service.
k apply -f np.yml

# Check the Services.
k get svc -o wide
```

```
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE     SELECTOR
drupal-mysql-service   ClusterIP   10.96.197.254   <none>        3306/TCP       30s     app=drupal-mysql
drupal-service         NodePort    10.96.106.84    <none>        80:30095/TCP   2m21s   app=drupal
kubernetes             ClusterIP   10.96.0.1       <none>        443/TCP        52m     <none>
```

Testing by clicking the App buttons shows it worked. We are done.
