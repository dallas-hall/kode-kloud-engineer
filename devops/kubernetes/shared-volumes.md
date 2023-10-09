# Shared Volumes

## Task

> We are working on an application that will be deployed on multiple containers within a pod on Kubernetes cluster. There is a requirement to share a volume among the containers to save some temporary data. The Nautilus DevOps team is developing a similar template to replicate the scenario. Below you can find more details about it.
>
> Create a pod named `volume-share-xfusion`.
>
> For the first container, use image fedora with latest tag only and remember to mention the tag i.e `fedora:latest`, container should be named as `volume-container-xfusion-1`, and run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/news`.
>
> For the second container, use image fedora with the latest tag only and remember to mention the tag i.e `fedora:latest` container should be named as `volume-container-xfusion-2`, and again run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/apps`.
>
> Volume name should be `volume-share` of type `emptyDir`.
>
> After creating the pod, `exec` into the first container i.e `volume-container-xfusion-1`, and just for testing create a file `news.txt` with any content under the mounted path of first container i.e `/tmp/news`.
>
> The file `news.txt` should be present under the mounted path `/tmp/apps` on the second container `volume-container-xfusion-2` as well, since they are using a shared volume.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Commands
  * https://kubernetes.io/docs/concepts/workloads/pods/
  * https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/
* emptyDir
  * https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
* exec
  * https://kubernetes.io/docs/reference/kubectl/cheatsheet/

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
# Create the Persistent Volume file
cat > pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-xfusion
spec:
  containers:
  - image: fedora:latest
    command: ["/bin/sh"]
    args: ["-c", "sleep 3600"]
    name: volume-container-xfusion-1
    volumeMounts:
    - mountPath: /tmp/news
      name: volume-share
  - image: fedora:latest
    name: volume-container-xfusion-2
    command: ["/bin/sh"]
    args: ["-c", "sleep 3600"]
    volumeMounts:
    - mountPath: /tmp/apps
      name: volume-share
  volumes:
  - name: volume-share
    emptyDir:
      sizeLimit: 128Mi
```

Close the file with control + d i.e. `^D`

```bash
# Create the Pod.
k apply -f pod.yml

# Check the Pod. It was running.
k get po -o wide

# Create the file.
k exec volume-share-xfusion -c volume-container-xfusion-1 -- bash -c 'echo "Hello world." > /tmp/news/news.txt'

# Check the file contents in both containers.
k exec volume-share-xfusion -c volume-container-xfusion-1 -- bash -c 'cat /tmp/news/news.txt'
k exec volume-share-xfusion -c volume-container-xfusion-2 -- bash -c 'cat /tmp/apps/news.txt'
```

Both returned `Hello world.`, we are done.
