# httpd Deployment

## Task

> There is an application that needs to be deployed on Kubernetes cluster under Apache web server. The Nautilus application development team has asked the DevOps team to deploy it. We need to develop a template as per requirements mentioned below:
>
> * Create a namespace named as `httpd-namespace-devops`.
> * Create a deployment named as `httpd-deployment-devops` under newly created namespace. For the deployment use `httpd` image with `latest` tag only and remember to mention the tag i.e `httpd:latest`, and make sure replica counts are `2`.
> * Create a service named as `httpd-service-devop` under same namespace to expose the deployment, nodePort should be `30004`.

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
NAME                      STATUS   ROLES           AGE     VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   8m18s   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Create the Namespace.
k create ns httpd-namespace-devops

# Create the Deployment.
k -n httpd-namespace-devops create deployment httpd-deployment-devops --image=httpd:latest --replicas=2 --port=80

# Create the NodePort Service YAML file.
cat > np-svc.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: httpd-deployment-devops
  name: httpd-service-devops
  namespace: httpd-namespace-devops
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30004
  selector:
    app: httpd-deployment-devops
  type: NodePort
```
```bash
# Create the NodePort Service.
k apply -f np-svc.yml

# Test the app.
curl http://kodekloud-control-plane:30004
```

```html
<html><body><h1>It works!</h1></body></html>
```

We are done.
