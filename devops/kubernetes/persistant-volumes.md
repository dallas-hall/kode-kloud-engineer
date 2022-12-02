# Persistent Volumes

## Task

> The Nautilus DevOps team is working on a Kubernetes template to deploy a web application on the cluster. There are some requirements to create/use persistent volumes to store the application code, and the template needs to be designed accordingly. Please find more details below:<br><br>Create a PersistentVolume named as `pv-xfusion`. Configure the spec as storage class should be `manual`, set capacity to `4Gi`, set access mode to `ReadWriteOnce`, volume type should be `hostPath` and set path to `/mnt/dba` (this directory is already created, you might not be able to access it directly, so you need not to worry about it).<br>Create a PersistentVolumeClaim named as `pvc-xfusion`. Configure the spec as storage class should be `manual`, request `2Gi` of the storage, set access mode to `ReadWriteOnce`.<br>Create a pod named as `pod-xfusion`, mount the persistent volume you created with claim name `pvc-xfusion` at document root of the web server, the container within the pod should be named as `container-xfusion` using image `httpd` with latest tag only (remember to mention the tag i.e httpd:latest).<br>Create a node port type service named `web-xfusion` using node port `30008` to expose the web server running within the pod.<br>**Note:** The kubectl utility on jump_host has been configured to work with the kubernetes cluster.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* PV and PVC - https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
* NodePort - https://kubernetes.io/docs/concepts/services-networking/service/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES                  AGE     VERSION                          INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane,master   3h35m   v1.20.5-rc.0.18+c4af4684437b37   172.17.0.2    <none>        Ubuntu 20.10   5.4.0-1093-gcp   containerd://1.5.0-beta.0-69-gb3f240206
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
  name: pv-xfusion
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/dba"
```

```bash
# Create the Persistent Volume Claim file
cat > pvc.yml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-xfusion
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```


```bash
# Create the PV and PVC object
k apply -f pv.yml && k apply -f pvc.yml
```

```
persistentvolume/pv-xfusion created
persistentvolumeclaim/pvc-xfusion created
```



```bash
# Check PV object
k get pv -o wide
```

```
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE     VOLUMEMODE
pv-xfusion   4Gi        RWO            Retain           Bound    default/pvc-xfusion   manual                  2m10s   Filesystem

```

```bash
# Check PVC object
k get pvc -o wide
```

```
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
pvc-xfusion   Bound    pv-xfusion   4Gi        RWO            manual         32s   Filesystem
```

```bash
# Create the Pod file
k run pod-xfusion --image=httpd:latest --dry-run -o yaml > pod.yml
vi pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-xfusion
  name: pod-xfusion
spec:
  containers:
  - image: httpd:latest
    name: container-xfusion
    ports:
    - containerPort: 80
      name: "http"
    - containerPort: 443
      name: "https"
    volumeMounts:
    - mountPath: "/var/www/html"
      name: my-mount
  volumes:
    - name: my-mount
      persistentVolumeClaim:
        claimName: pvc-xfusion
  restartPolicy: Always
```

```bash
# Create & check the pod
k apply -f pod.yaml && k get po -o wide
```

```
pod/pod-xfusion created
NAME          READY   STATUS    RESTARTS   AGE   IP           NODE                      NOMINATED NODE   READINESS GATES
pod-xfusion   1/1     Running   0          43s   10.244.0.5   kodekloud-control-plane   <none>           <none>
```

```bash
# Create the NodePort file
k expose pod pod-xfusion --name=web-xfusion --type=NodePort --dry-run -o yaml > nodeport.yml
vi nodeport.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    run: pod-xfusion
  name: web-xfusion
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
    run: pod-xfusion
  type: NodePort
status:
```

```bash
# Create and check the NodePort
k apply -f nodeport.yml && k get svc -o wide
```

```
service/web-xfusion created
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE     SELECTOR
kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP                      3h36m   <none>
web-xfusion   NodePort    10.96.203.190   <none>        80:30008/TCP,443:30009/TCP   9s      run=pod-xfusion
```

```bash
# Test the app's external connection
curl http://kodekloud-control-plane:30008
```

```html
<html><body><h1>It works!</h1></body></html>
```

Works, we are done.