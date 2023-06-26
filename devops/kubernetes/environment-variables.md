# Environment Variables

## Task

> There are a number of parameters that are used by the applications. We need to define these as environment variables, so that we can use them as needed within different configs. Below is a scenario which needs to be configured on Kubernetes cluster. Please find below more details about the same.<br><br>Create a pod named `envars`.<br>Container name should be `fieldref-container`, use image `nginx` preferable `latest` tag, use `command` `'sh', '-c'` and `args` should be `'while true; do echo -en '/n'; printenv NODE_NAME POD_NAME; printenv POD_IP POD_SERVICE_ACCOUNT; sleep 10; done;'`(Note: please take care of indentations)<br><br>Define Four environment variables as mentioned below:<br>a.) The first env should be named as `NODE_NAME`, set valueFrom fieldref and fieldPath should be spec.nodeName.<br>b.) The second env should be named as `POD_NAME`, set valueFrom fieldref and fieldPath should be metadata.name.<br>c.) The third env should be named as `POD_IP`, set valueFrom fieldref and fieldPath should be status.podIP.<br>d.) The fourth env should be named as `POD_SERVICE_ACCOUNT`, set valueFrom fieldref and fieldPath shoulbe be spec.serviceAccountName.<br> Set restart policy to Never.<br>To check the output, exec into the pod and use `printenv` command.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

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
NAME                      STATUS   ROLES                  AGE    VERSION                          INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane,master   145m   v1.20.5-rc.0.18+c4af4684437b37   172.17.0.2    <none>        Ubuntu 20.10   5.4.0-1092-gcp   containerd://1.5.0-beta.0-69-gb3f240206
```

```bash
# Create the Pod YAML file
cat > pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envars
spec:
  containers:
    - name: fieldref-container
      image: nginx:latest
      command: ["sh", "-c"]
      args:
      - "while true; do echo -en '/n'; printenv NODE_NAME POD_NAME; printenv POD_IP POD_SERVICE_ACCOUNT; sleep 10; done;"
      env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
  restartPolicy: Never
```

```bash
# Creat the Pod
k apply -f pod.yaml
```

```
pod/envars created
```

```bash
# View the environment variables
k exec -it envars -- printenv
```

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=envars
NGINX_VERSION=1.25.1
NJS_VERSION=0.7.12
PKG_RELEASE=1~bookworm
NODE_NAME=kodekloud-control-plane
POD_NAME=envars
POD_IP=10.244.0.5
POD_SERVICE_ACCOUNT=default
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
TERM=xterm
HOME=/root
```

We are done.