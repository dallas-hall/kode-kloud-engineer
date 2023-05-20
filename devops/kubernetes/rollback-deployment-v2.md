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
NAME                      STATUS   ROLES                  AGE    VERSION                          INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane,master   145m   v1.20.5-rc.0.18+c4af4684437b37   172.17.0.2    <none>        Ubuntu 20.10   5.4.0-1092-gcp   containerd://1.5.0-beta.0-69-gb3f240206
```

We can connect fine from Thor jumpbox.

```bash
# Get Deployment
k get deployments.apps
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           2m30s
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
k get po
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-74fb588559-5fkns   1/1     Running   0          37s
nginx-deployment-74fb588559-hndd9   1/1     Running   0          32s
nginx-deployment-74fb588559-kjfsz   1/1     Running   0          35s
```

New Pod's have been created.

```bash
# See if there is a NodePort Service for testing
k get svc
```

```
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        90m
nginx-service   NodePort    10.96.105.201   <none>        80:30008/TCP   4m7s
```

There is a NodePort Service that we can use for testing.

```bash
# Get the Node's IP
nslookup kodekloud-control-plane
```

```
Server:         127.0.0.11
Address:        127.0.0.11#53

Non-authoritative answer:
Name:   kodekloud-control-plane
Address: 172.18.0.5
```

```bash
# Test the Deployment
curl 172.18.0.5:30008
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
</html>
```

</details>

We are done.