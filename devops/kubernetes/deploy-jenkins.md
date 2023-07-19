# Jenkins Deployment

## Task

> The Nautilus DevOps team is planning to set up a Jenkins CI server to create/manage some deployment pipelines for some of the projects. They want to set up the Jenkins server on Kubernetes cluster. Below you can find more details about the task:<br><br>1) Create a namespace `jenkins`<br>2) Create a Service for jenkins deployment. Service name should be `jenkins-service` under jenkins namespace, type should be NodePort, nodePort should be `30008`<br>3) Create a Jenkins Deployment under jenkins namespace, It should be name as `jenkins-deployment`, labels `app` should be `jenkins`, container name should be `jenkins-container`, use `jenkins/jenkins` image , `containerPort` should be `8080` and replicas count should be 1.<br>Make sure to wait for the pods to be in running state and make sure you are able to access the Jenkins login screen in the browser before hitting the Check button.

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
# Create the namespace
k create ns jenkins
```

```
namespace/jenkins created
```

```bash
# Create the deployment
cat > deploy.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: jenkins
  name: jenkins-deployment
  namespace: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - image: jenkins/jenkins
        name: jenkins-container
        ports:
        - containerPort: 8080
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
    app: jenkins
  name: jenkins-service
  namespace: jenkins
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
    nodePort: 30008
  selector:
    app: jenkins
  type: NodePort
```

Close the file with control + d i.e. `^D`

```bash
k apply -f deploy.yaml
```

```
service/jenkins-service created
```

We are done.