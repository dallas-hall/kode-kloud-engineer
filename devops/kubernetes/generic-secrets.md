# Deployment Rollout

## Task

> The Nautilus DevOps team is working to deploy some tools in Kubernetes cluster. Some of the tools are licence based so that licence information needs to be stored securely within Kubernetes cluster. Therefore, the team wants to utilize Kubernetes secrets to store those secrets. Below you can find more details about the requirements:
> * We already have a secret key file `beta.txt` under `/opt` location on jump host. Create a generic secret named `beta`, it should contain the password/license-number present in `beta.txt` file.
> * Also create a pod named `secret-devops`.
> * Configure pod's spec as container name should be `secret-container-devops`, image should be `debian` preferably with `latest` tag (remember to mention the tag with image). Use `sleep` command for container so that it remains in running state. Consume the created secret and mount it under `/opt/games` within the container.
> * To verify you can `exec` into the container `secret-container-devops`, to check the secret key under the mounted path `/opt/games`.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Commands.
  * https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/
* Secrets.
  * https://kubernetes.io/docs/concepts/configuration/secret/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   95m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Create the Secret.
k create secret generic beta --from-file=/opt/beta.txt

# Create the Pod YAML file
cat > pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: secret-devops
  name: secret-devops
spec:
  containers:
  - image: debian:latest
    name: secret-container-devops
    volumeMounts:
    - name: foo
      mountPath: "/opt/games"
      readOnly: true
    command: ["sleep"]
    args: ["999"]
  volumes:
  - name: foo
    secret:
      secretName: beta
```

Close the file with control + d i.e. `^D`

```bash
# Create the pod
k apply -f pod.yaml

# Check the secret
k exec -it secret-devops -- cat /opt/games/beta.txt
```

```
5ecur3
```

We are done.
