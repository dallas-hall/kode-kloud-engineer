# Create A Namespace

## Task

> The Nautilus DevOps team is planning to deploy some micro services on Kubernetes platform. The team has already set up a Kubernetes cluster and now they want set up some namespaces, deployments etc. Based on the current requirements, the team has shared some details as below:<br><br>Create a namespace named `dev` and create a POD under it; name the pod `dev-nginx-pod` and use nginx image with latest tag only and remember to mention tag i.e `nginx:latest`.

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
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   12m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# View namespaces
k get ns
```

```
NAME                 STATUS   AGE
default              Active   11m
kube-node-lease      Active   11m
kube-public          Active   11m
kube-system          Active   11m
local-path-storage   Active   10m
```

```bash
# Create the namespace.
k create ns dev
```

```
namespace/dev created
```


```bash
# Create the Pod.
k -n dev run dev-nginx-pod --image=nginx:latest --port=80
```

```
pod/dev-nginx-pod created
```

```bash
# Expose the Pod.
k -n dev expose pod dev-nginx-pod --port=80 --type=NodePort --name=external-dev-nginx-pod
```

```
service/external-dev-nginx-pod exposed
```

```bash
# View the Pod.
k -n dev get po
```

```
NAME            READY   STATUS    RESTARTS   AGE
dev-nginx-pod   1/1     Running   0          29s
```

```bash
# View the Service.
k -n dev get svc
```

```
NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
external-dev-nginx-pod   NodePort   10.96.200.240   <none>        80:30780/TCP   29s
```

```bash
# Test external connectivity.
curl kodekloud-control-plane:30780
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
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
