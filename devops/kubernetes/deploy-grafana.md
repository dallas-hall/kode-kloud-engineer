# Grafana Deployment

## Task

> The Nautilus DevOps teams is planning to set up a Grafana tool to collect and analyze analytics from some applications. They are planning to deploy it on Kubernetes cluster. Below you can find more details.
>
> * Create a deployment named `grafana-deployment-xfusion` using any `grafana` image for Grafana app. Set other parameters as per your choice.
> * Create NodePort type service with nodePort `32000` to expose the app.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Grafana images
  * https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/
* Grafana ports
  * https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE    VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   145m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Create the deployment
cat > deploy.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana
  name: grafana-deployment-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - image: grafana/grafana-oss
        name: grafana-container
        ports:
        - containerPort: 3000
```

Close the file with control + d i.e. `^D`

```bash
k apply -f deploy.yaml
```

```
deployment.apps/jenkins-deployment created
```

```bash
# Create the NodePort Service
cat > svc.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: grafana
  name: grafana-external-service
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 3000
    nodePort: 32000
  selector:
    app: grafana
  type: NodePort
```

Close the file with control + d i.e. `^D`

```bash
k apply -f svc.yaml
```

```
service/jenkins-service created
```

```bash
# Test with curl
curl -L -v kodekloud-control-plane:32000
```

Pages of HTML was return. Clicking the Jenkins button brings up the Grafana UI.

We are done.
