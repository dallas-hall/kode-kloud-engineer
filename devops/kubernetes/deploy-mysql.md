# MySQL Deployment

## Task

> A new MySQL server needs to be deployed on Kubernetes cluster. The Nautilus DevOps team was working on to gather the requirements. Recently they were able to finalize the requirements and shared them with the team members to start working on it. Below you can find the details:
>
> * Create a PersistentVolume `mysql-pv`, its capacity should be `250Mi`, set other parameters as per your preference.
> * Create a PersistentVolumeClaim to request this PersistentVolume storage. Name it as `mysql-pv-claim` and request a `250Mi` of storage. Set other parameters as per your preference.
> * Create a deployment named `mysql-deployment`, use any `mysql` image as per your preference. Mount the PersistentVolume at mount path `/var/lib/mysql`.
> * Create a NodePort type service named `mysql` and set `nodePort` to 30007.
> * Create a secret named `mysql-root-pass` having a key pair value, where key is `password` and its value is `YUIidhb667`, create another secret named `mysql-user-pass` having some key pair values, where frist key is `username` and its value is `kodekloud_roy`, second key is `password` and value is `8FmzjvFU6S`, create one more secret named `mysql-db-url`, key name is `database` and value is `kodekloud_db1`
> * Define some Environment variables within the container:
>   * name: `MYSQL_ROOT_PASSWORD`, should pick value from `secretKeyRef` `name: mysql-root-pass` and `key: password`
>   * name: `MYSQL_DATABASE`, should pick value from `secretKeyRef` `name: mysql-db-url` and `key: database`
>   * name: `MYSQL_USER`, should pick value from `secretKeyRef` `name: mysql-user-pass` and key `key: username`
>   * name: `MYSQL_PASSWORD`, should pick value from `secretKeyRef` `name: mysql-user-pass` and `key: password`

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* PV and PVC.
  * https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
* NodePort.
  * https://kubernetes.io/docs/concepts/services-networking/service/
* MySQL
  * https://hub.docker.com/_/mysql

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   75m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Create the Persistent Volume file
cat > pv.yml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
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
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
```

Close the file with control + d i.e. `^D`

```bash
# Check the binding, it was okay.
k get pv,pvc
```

```
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim   manual                  54s

NAME             STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    mysql-pv   250Mi      RWO            manual         30s
```

```bash
# Create Secrets.
k create secret generic mysql-root-pass --from-literal password=YUIidhb667
k create secret generic mysql-user-pass --from-literal username=kodekloud_roy --from-literal password=8FmzjvFU6S
k create secret generic mysql-db-url --from-literal database=kodekloud_db1
```

```bash
# Create Deployment YAML.
cat > d.yml
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mysql-deployment
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql-deployment
  template:
    metadata:
      labels:
        app: mysql-deployment
    spec:
      containers:
      - image: mysql
        name: mysql
        ports:
        - containerPort: 3306
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: data
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-pass
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-db-url
              key: database
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: password
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: mysql-pv-claim
```

```bash
# Create the Deployment.
k apply -f d.yml

# Wait for the Deployment to be ready.
k get deployments.apps -w
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
mysql-deployment   0/1     1            0           8s
mysql-deployment   1/1     1            1           48s
```


```bash
# Create the NodePort YAML.
cat > np.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: mysql-deployment
  name: mysql
spec:
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
    nodePort: 30007
  selector:
    app: mysql-deployment
  type: NodePort
```

Close the file with control + d i.e. `^D`

```bash
# Deploy the Service
k apply -f np.yml

# Check the NodePort service.
k get svc
```

```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          38m
mysql        NodePort    10.96.133.158   <none>        3306:30007/TCP   4s
```

We are done.
