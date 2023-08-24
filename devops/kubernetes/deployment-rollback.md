# Deployment Rollbacks

## Task

> This morning the Nautilus DevOps team rolled out a new release for one of the applications. Recently one of the customers logged a complaint which seems to be about a bug related to the recent release. Therefore, the team wants to rollback  the recent release. There is a deployment named `nginx-deployment`; roll it back to the previous revision.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* N/A

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE     VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   6m56s   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Get Deployment
k get deploy
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           2m49s
```

```bash
# View Deployment Rollout history
k rollout history deployment nginx-deployment
```

```
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deployment nginx-container=nginx:alpine-perl --kubeconfig=/root/.kube/config --record=true
```

```bash
# Roll back the Deployment
k rollout undo deployment nginx-deployment
```

```
deployment.apps/nginx-deployment rolled back
```

```bash
# Check the new Pods
k get po
```

```
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-989f57c54-bmxxq   1/1     Running   0          6s
nginx-deployment-989f57c54-btljn   1/1     Running   0          8s
nginx-deployment-989f57c54-rdmjl   1/1     Running   0          9s
```

New Pod's have been created.

```bash
# See if there is a NodePort Service for testing
k get svc
```

```
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        7m53s
nginx-service   NodePort    10.96.14.144   <none>        80:30008/TCP   3m35s
```

There is a NodePort Service that we can use for testing.

```bash
# Test the Deployment
curl curl kodekloud-control-plane:30008
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
```

</details>

We are done.
