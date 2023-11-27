# Tomcat Deployment

## Task

> A new java-based application is ready to be deployed on a Kubernetes cluster. The development team had a meeting with the DevOps team to share the requirements and application scope. The team is ready to setup an application stack for it under their existing cluster. Below you can find the details for this:
>
> * Create a namespace named `tomcat-namespace-datacenter`.
> * Create a deployment for tomcat app which should be named as `tomcat-deployment-datacenter` under the same namespace you created. Replica count should be 1, the container should be named as `tomcat-container-datacenter`, its image should be `gcr.io/kodekloud/centos-ssh-enabled:tomcat` and its container port should be `8080`.
> * Create a service for tomcat app which should be named as `tomcat-service-datacenter` under the same namespace you created. Service type should be `NodePort` and nodePort should be `32227`.

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
kodekloud-control-plane   Ready    control-plane   68m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Create the namespace.
k create ns tomcat-namespace-datacenter

# Create the deployment YAML file
cat > deploy.yml
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: tomcat-deployment-datacenter
  name: tomcat-deployment-datacenter
  namespace: tomcat-namespace-datacenter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat-deployment-datacenter
  template:
    metadata:
      labels:
        app: tomcat-deployment-datacenter
    spec:
      containers:
      - image: gcr.io/kodekloud/centos-ssh-enabled:tomcat
        name: tomcat-container-datacenter
        ports:
        - containerPort: 8080
```

Close the file with control + d i.e. `^D`

```bash
# Create the deployment
k apply -f deploy.yml

# Create the NodePort YAML file
cat > nodeport.yml
```

```yml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: tomcat-deployment-datacenter
  name: tomcat-service-datacenter
  namespace: tomcat-namespace-datacenter
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
    nodePort: 32227
    name: http
  selector:
    app: tomcat-deployment-datacenter
  type: NodePort
```

Close the file with control + d i.e. `^D`

```bash
# Create the NodePort
k apply -f nodeport.yml

# Test connectivity
curl kodekloud-control-plane:32227
```

Clicking the App button in the UI provides the same result.

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```html
<html>
    <head>
        <title>SampleWebApp</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </head>
    <body>
        <h2>Welcome to xFusionCorp Industries!</h2>
        <br>
    
    </body>
</html>
```

</details>

We are done.
