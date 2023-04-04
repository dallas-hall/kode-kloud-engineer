# Deployment Rollout

## Task

> The Nautilus DevOps team is working to deploy some tools in Kubernetes cluster. Some of the tools are licence based so that licence information needs to be stored securely within Kubernetes cluster. Therefore, the team wants to utilize Kubernetes secrets to store those secrets. Below you can find more details about the requirements:<br><br>We already have a secret key file `news.txt` under `/opt` location on jump host. Create a generic secret named news, it should contain the password/license-number present in `news.txt` file.<br>Also create a pod named `secret-xfusion`.<br>Configure pod's spec as container name should be `secret-container-xfusion`, image should be `centos` preferably with `latest` tag (remember to mention the tag with image). Use `sleep` command for container so that it remains in running state. Consume the created secret and mount it under `/opt/`apps within the container.<br>To verify you can `exec` into the container `secret-container-xfusion`, to check the secret key under the mounted path `/opt/apps`.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Commands - https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/
* Secrets - https://kubernetes.io/docs/concepts/configuration/secret/

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

```bash
# Create the secret
k create secret generic news --from-file=/opt/news.txt
secret/news created

# Create the Pod YAML file
cat > pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: secret-xfusion
  name: secret-xfusion
spec:
  containers:
  - image: centos:latest
    name: secret-container-xfusion
    volumeMounts:
    - name: foo
      mountPath: "/opt/apps"
      readOnly: true
    command: ["sleep"]
    args: ["999"]
  volumes:
  - name: foo
    secret:
      secretName: news
```

Close the file with control + d i.e. `^D`

```bash
# Create the pod
k apply -f pod.yaml
pod/secret-xfusion created

# Check the secret
k exec -it secret-xfusion -- cat /opt/apps/news.txt
```

```
5ecur3
```

We are done.