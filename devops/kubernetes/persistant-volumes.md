# Persistent Volumes

## Task

> The Nautilus DevOps team is working on a Kubernetes template to deploy a web application on the cluster. There are some requirements to create/use persistent volumes to store the application code, and the template needs to be designed accordingly. Please find more details below:
> 
> * Create a PersistentVolume named as `pv-devops`. Configure the spec as storage class should be `manual`, set capacity to `5Gi`, set access mode to `ReadWriteOnce`, volume type should be `hostPath` and set path to `/mnt/devops` (this directory is already created, you might not be able to access it directly, so you need not to worry about it).
> * Create a PersistentVolumeClaim named as `pvc-devops`. Configure the spec as storage class should be `manual`, request `1Gi` of the storage, set access mode to `ReadWriteOnce`.
> * Create a pod named as `pod-devops`, mount the persistent volume you created with claim name `pvc-devops` at document root of the web server, the container within the pod should be named as `container-devops` using image `nginx` with latest tag only (remember to mention the tag i.e `nginx:latest`).
> * Create a node port type service named `web-devops` using node port `30008` to expose the web server running within the pod.


## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* PV and PVC.
  * https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
* NodePort.
  * https://kubernetes.io/docs/concepts/services-networking/service/
* nginx document root path
  * https://stackoverflow.com/a/14197550

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   81m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
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
  name: pv-devops
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/devops"
```

Close the file with control + d i.e. `^D`


```bash
# Create the Persistent Volume Claim file
cat > pvc.yml
```

Close the file with control + d i.e. `^D`


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-devops
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```


```bash
# Create the PV and PVC object
k apply -f pv.yml
k apply -f pvc.yml

# Check PV object
k get pv -o wide
```

```
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE   VOLUMEMODE
pv-devops   5Gi        RWO            Retain           Bound    default/pvc-devops   manual                  14s   Filesystem
```

Looks good.

```bash
# Check PVC object
k get pvc -o wide
```

```
NAME         STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
pvc-devops   Bound    pv-devops   5Gi        RWO            manual         25s   Filesystem
```

Looks good.

```bash
# Create the Pod file
cat > pod.yml
```

Close the file `:x`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-devops
  name: pod-devops
spec:
  containers:
  - image: nginx:latest
    name: container-devops
    ports:
    - containerPort: 80
      name: "http"
    - containerPort: 443
      name: "https"
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: my-mount
  volumes:
    - name: my-mount
      persistentVolumeClaim:
        claimName: pvc-devops
  restartPolicy: Always
```

Close the file `:x`

```bash
# Create & check the Pod, it was ok.
k apply -f pod.yaml
k get po -o wide

# Create the NodePort file
cat > np.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    run: pod-devops
  name: web-devops
spec:
  ports:
  - name: port-1
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30008
  - name: port-2
    port: 443
    protocol: TCP
    targetPort: 443
    nodePort: 30009
  selector:
    run: pod-devops
  type: NodePort
```

Close the file `:x`

```bash
# Create and check the NodePort
k apply -f nodeport.yml
k get svc -o wide
```

```
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE   SELECTOR
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP                      88m   <none>
web-devops   NodePort    10.96.141.67   <none>        80:30008/TCP,443:30009/TCP   26s   run=pod-devops
```

```bash
# Test the app's external connection
curl http://kodekloud-control-plane:30008
```

```html
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.25.3</center>
</body>
</html>
```

Connected to the container and there were no files inside of that folder so there was nothing to display. We are done.
