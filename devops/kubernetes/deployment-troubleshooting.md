# Deployment Troubleshooting

## Task

> Last week, the Nautilus DevOps team deployed a redis app on Kubernetes cluster, which was working fine so far. This morning one of the team members was making some changes in this existing setup, but he made some mistakes and the app went down. We need to fix this as soon as possible. Please take a look.<br><br>The deployment name is `redis-deployment`. The pods are not in running state right now, so please look into the issue and fix the same.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

None.

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

We can connect fine from Thor jumpbox.

```bash
# Check the Deployment
k get deploy
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   0/1     1
```

The Deployment has a broken Pod.

```bash
# Check the ReplicaSet
k get rs
```

```
NAME                         DESIRED   CURRENT   READY   AGE
redis-deployment-76cc9c56f   1         1         0       10m
```

The ReplicaSet has a broken Pod.

```bash
# Check the Pod
k get po
```

```
NAME                               READY   STATUS              RESTARTS   AGE
redis-deployment-76cc9c56f-96vxr   0/1     ContainerCreating   0          10m
```

The Pod has a problem while being created.

```bash
# Inspect the Pod
k describe po redis-deployment-76cc9c56f-96vxr
```

```
...
Events:
  Type     Reason       Age                    From               Message
  ----     ------       ----                   ----               -------
  Normal   Scheduled    11m                    default-scheduler  Successfully assigned default/redis-deployment-76cc9c56f-96vxr to kodekloud-control-plane
  Warning  FailedMount  2m30s (x3 over 9m18s)  kubelet            Unable to attach or mount volumes: unmounted volumes=[config], unattached volumes=[data config default-token-9n9jf]: timed out waiting for the condition
  Warning  FailedMount  65s (x13 over 11m)     kubelet            MountVolume.SetUp failed for volume "config" : configmap "redis-cofig" not found
  Warning  FailedMount  13s (x2 over 4m48s)    kubelet            Unable to attach or mount volumes: unmounted volumes=[config], unattached volumes=[default-token-9n9jf data config]: timed out waiting for the condition
```

There is an issue with a ConfigMap being mounted as a Volume. Looks like a typo.

```bash
# View ConfigMap
k get cm
```

```
NAME               DATA   AGE
kube-root-ca.crt   1      4h27m
redis-config       2      19m
```

```bash
# Back up the Deployment
k get deploy -o yaml > deploy.yaml

# Fix the typo in the ConfigMap
k edit deploy redis-deployment
```

```
deployment.apps/redis-deployment edited
```

```bash
# Check the Pod
k get po
```

```
NAME                                READY   STATUS             RESTARTS   AGE
redis-deployment-6589b5b779-hdgq7   0/1     ImagePullBackOff   0          53s
```

There is a problem with the image.

```bash
# Inspect the Pod
k describe po redis-deployment-6589b5b779-hdgq7
```

```
...
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  73s                default-scheduler  Successfully assigned default/redis-deployment-6589b5b779-hdgq7 to kodekloud-control-plane
  Normal   Pulling    33s (x3 over 72s)  kubelet            Pulling image "redis:alpin"
  Warning  Failed     33s (x3 over 72s)  kubelet            Failed to pull image "redis:alpin": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/redis:alpin": failed to resolve reference "docker.io/library/redis:alpin": docker.io/library/redis:alpin: not found
  Warning  Failed     33s (x3 over 72s)  kubelet            Error: ErrImagePull
  Normal   BackOff    6s (x4 over 71s)   kubelet            Back-off pulling image "redis:alpin"
  Warning  Failed     6s (x4 over 71s)   kubelet            Error: ImagePullBackOff
```

There is a typo in the image name.

```bash
# Fix the typo in the image name.
k edit deploy redis-deployment
```

```
deployment.apps/redis-deployment edited
```

```bash
# Check the Pod
k get po
```

```
NAME                                READY   STATUS    RESTARTS   AGE
redis-deployment-5bb6dd57fd-v56ms   1/1     Running   0          46s
```

Looks good.
