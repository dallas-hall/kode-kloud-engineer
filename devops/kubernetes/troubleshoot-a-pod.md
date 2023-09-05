# Troubleshoot A Pod

## Task

> One of the junior DevOps team members was working on to deploy a stack on Kubernetes cluster. Somehow the pod is not coming up and its failing with some errors. We need to fix this as soon as possible. Please look into it.
>
> * There is a pod named `webserver` and the container under it is named as `httpd-container`. It is using image `httpd:latest`
>
> * There is a sidecar container as well named `sidecar-container` which is using `ubuntu:latest` image.
>
> Look into the issue and fix it, make sure pod is in running state and you are able to access the app.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* CronJobs
  * https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
  * https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE     VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   6m34s   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Check the Pod's status.
k get po -o wide
```

```
NAME        READY   STATUS             RESTARTS   AGE    IP           NODE                      NOMINATED NODE   READINESS GATES
webserver   1/2     ImagePullBackOff   0          2m7s   10.244.0.5   kodekloud-control-plane   <none>           <none>
```

There is a problem with one of the images.

```bash
# Inspect the Pod.
k describe po
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Name:             webserver
Namespace:        default
Priority:         0
Service Account:  default
Node:             kodekloud-control-plane/172.17.0.2
Start Time:       Tue, 05 Sep 2023 08:11:26 +0000
Labels:           app=web-app
Annotations:      <none>
Status:           Pending
IP:               10.244.0.5
IPs:
  IP:  10.244.0.5
Containers:
  httpd-container:
    Container ID:
    Image:          httpd:latests
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log/httpd from shared-logs (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-nvjlw (ro)
  sidecar-container:
    Container ID:  containerd://265da5444007674521b2844caecb6ea1780b7bec83793a3de0cc091694265555
    Image:         ubuntu:latest
    Image ID:      docker.io/library/ubuntu@sha256:aabed3296a3d45cede1dc866a24476c4d7e093aa806263c27ddaadbdce3c1054
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      while true; do cat /var/log/httpd/access.log /var/log/httpd/error.log; sleep 30; done
    State:          Running
      Started:      Tue, 05 Sep 2023 08:11:32 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log/httpd from shared-logs (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-nvjlw (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  shared-logs:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  kube-api-access-nvjlw:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Normal   Scheduled  3m17s                  default-scheduler  Successfully assigned default/webserver to kodekloud-control-plane
  Normal   Pulling    3m16s                  kubelet            Pulling image "ubuntu:latest"
  Normal   Pulled     3m11s                  kubelet            Successfully pulled image "ubuntu:latest" in 4.833482466s (4.833505526s including waiting)
  Normal   Created    3m11s                  kubelet            Created container sidecar-container
  Normal   Started    3m11s                  kubelet            Started container sidecar-container
  Normal   Pulling    2m24s (x3 over 3m16s)  kubelet            Pulling image "httpd:latests"
  Warning  Failed     2m23s (x3 over 3m16s)  kubelet            Failed to pull image "httpd:latests": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/httpd:latests": failed to resolve reference "docker.io/library/httpd:latests": docker.io/library/httpd:latests: not found
  Warning  Failed     2m23s (x3 over 3m16s)  kubelet            Error: ErrImagePull
  Normal   BackOff    107s (x6 over 3m10s)   kubelet            Back-off pulling image "httpd:latests"
  Warning  Failed     107s (x6 over 3m10s)   kubelet            Error: ImagePullBackOff
```

</details>

We can see in the output the problem is "*Failed to pull image "httpd:latests": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/httpd:latests": failed to resolve reference "docker.io/library/httpd:latests": docker.io/library/httpd:latests: not found*".

```bash
# Update the image.
k set image po/webserver httpd-container=httpd:latest
```

```
pod/webserver image updated
```

```bash
# Check the Pod's status.
k get po -o wide
```

```
NAME        READY   STATUS    RESTARTS   AGE     IP           NODE                      NOMINATED NODE   READINESS GATES
webserver   2/2     Running   0          7m17s   10.244.0.5   kodekloud-control-plane   <none>           <none>
```

We are done.
