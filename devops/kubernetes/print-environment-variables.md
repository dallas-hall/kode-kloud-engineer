# Print Environment Varaibles

## Task

> The Nautilus DevOps team is working on to setup some pre-requisites for an application that will send the greetings to different users. There is a sample deployment, that needs to be tested. Below is a scenario which needs to be configured on Kubernetes cluster. Please find below more details about it.
>
> * Create a pod named `print-envars-greeting`.
> * Configure spec as, the container name should be `print-env-container` and use `bash` image.
> * Create three environment variables:
>   * `GREETING`and its value should be `Welcome to`
>   * `COMPANY`and its value should be `Stratos`
>   * `GROUP` and its value should be `Ltd`
> * Use command to `echo ["$(GREETING) $(COMPANY) $(GROUP)"]` message.
> * You can check the output using `kubectl logs -f print-envars-greeting` command.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Pods
  * https://kubernetes.io/docs/concepts/workloads/pods/
* Commands.
  * https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/
* Environment Variables.
  * https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/
  * https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   36m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Create Pod YAML
cat > pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: print-envars-greeting
  name: print-envars-greeting
spec:
  containers:
  - image: bash:latest
    name: print-env-container
    command: ["sh", "-c"]
    args: ["echo $(GREETING) $(COMPANY) $(GROUP)"]
    env:
    - name: GREETING
      value: "Welcome to"
    - name: COMPANY
      value: "Stratos"
    - name: GROUP
      value: "Ltd"

```
Close the file with control + d i.e. `^D`


```bash
# Create the Pod
k apply -f  pod.yaml

# Check the Pod, it was OK.
k get po

# Check the logs.
k logs -f print-envars-greeting
```

```
Welcome to Stratos Ltd
```

We are done.
