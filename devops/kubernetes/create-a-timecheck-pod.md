# Create A CronJob

## Task

>The Nautilus DevOps team want to create a time check pod in a particular Kubernetes namespace and record the logs. This might be initially used only for testing purposes, but later can be implemented in an existing cluster. Please find more details below about the task and perform it.
>
> * Create a pod called `time-check` in the `xfusion` namespace. This pod should run a container called `time-check`, container should use the busybox image with latest tag only and remember to mention tag i.e `busybox:latest`.
> * Create a config map called `time-config` with the data `TIME_FREQ=2` in the same namespace.
> * The time-check container should run the command:`` while true; do date; sleep $TIME_FREQ;done` and should write the result to the location `/opt/security/time/time-check.log`. Remember you will also need to add an environmental variable TIME_FREQ in the container, which should pick value from the config map TIME_FREQ key.
> * Create a volume `log-volume` and mount the same on `/opt/security/time` within the container.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* ConfigMaps
  * https://kubernetes.io/docs/concepts/configuration/configmap/
  * https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
* Volumes
  * https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
* Pods
  * https://kubernetes.io/docs/concepts/workloads/pods/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   25m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Check Namespaces
k get ns
```

```
NAME                 STATUS   AGE
default              Active   25m
kube-node-lease      Active   25m
kube-public          Active   25m
kube-system          Active   25m
local-path-storage   Active   25m
```

```bash
# Create the Namespace.
k create ns xfusion
```

```
namespace/xfusion created
```

```bash
# Create ConfigMap
k create cm -n xfusion time-config --from-literal='TIME_FREQ=2
```

```
configmap/time-config created
```

```bash
# Create Pod YAML
cat > pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: time-check
  name: time-check
  namespace: xfusion
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - while true; do date; sleep $TIME_FREQ;done &> /opt/security/time/time-check.log
    image: busybox:latest
    name: time-check
    env:
    - name: TIME_FREQ
      valueFrom:
        configMapKeyRef:
          name: time-config
          key: TIME_FREQ
    volumeMounts:
    - mountPath: /opt/security/time
      name: log-volume
  volumes:
  - name: log-volume
    emptyDir:
      sizeLimit: 512Mi
```

Close the file with control + d i.e. `^D`

```bash
# Create the Pod
k apply -f pod.yaml
```

```
pod/time-check created
```

```bash
# Check the Pod
k exec -n xfusion time-check -- cat /opt/security/time/time-check.log
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Mon Sep  4 08:53:08 UTC 2023
Mon Sep  4 08:53:10 UTC 2023
Mon Sep  4 08:53:12 UTC 2023
Mon Sep  4 08:53:14 UTC 2023
Mon Sep  4 08:53:16 UTC 2023
Mon Sep  4 08:53:18 UTC 2023
Mon Sep  4 08:53:20 UTC 2023
Mon Sep  4 08:53:22 UTC 2023
Mon Sep  4 08:53:24 UTC 2023
Mon Sep  4 08:53:26 UTC 2023
Mon Sep  4 08:53:28 UTC 2023
Mon Sep  4 08:53:30 UTC 2023
Mon Sep  4 08:53:32 UTC 2023
Mon Sep  4 08:53:34 UTC 2023
Mon Sep  4 08:53:36 UTC 2023
Mon Sep  4 08:53:38 UTC 2023
Mon Sep  4 08:53:40 UTC 2023
Mon Sep  4 08:53:42 UTC 2023
Mon Sep  4 08:53:44 UTC 2023
Mon Sep  4 08:53:46 UTC 2023
Mon Sep  4 08:53:48 UTC 2023
Mon Sep  4 08:53:50 UTC 2023
Mon Sep  4 08:53:52 UTC 2023
Mon Sep  4 08:53:54 UTC 2023
Mon Sep  4 08:53:56 UTC 2023
Mon Sep  4 08:53:58 UTC 2023
Mon Sep  4 08:54:00 UTC 2023
Mon Sep  4 08:54:02 UTC 2023
Mon Sep  4 08:54:05 UTC 2023
```

</details>

We are done.
