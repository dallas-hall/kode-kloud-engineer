# Resource Limits

## Task

> Recently some of the performance issues were observed with some applications hosted on Kubernetes cluster. The Nautilus DevOps team has observed some resources constraints, where some of the applications are running out of resources like memory, cpu etc., and some of the applications are consuming more resources than needed. Therefore, the team has decided to add some limits for resources utilization. Below you can find more details.<br><br>Create a pod named`httpd-pod` and a container under it named as `httpd-container`, use httpd image with latest tag only and remember to mention tag i.e `httpd:latest` and set the following limits:<br>Requests: Memory: `15Mi`, CPU: `100m`br>Limits: Memory: `20Mi`, CPU: `100m`

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Resource limits
  * https://kubernetes.io/docs/concepts/policy/limit-range/
  * https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/
  * https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE     VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   8m36s   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Create CPU limit YAML file
cat > ~/cpu-limit.yml
```

```yml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - min:
      cpu: 10m # 10m = 1% of 1 hyperthread
    max:
      cpu: 1000m # 1000m = 1 hyperthread
    default:
      cpu: 1000m
    defaultRequest:
      cpu: 500m # 500m = 50% of 1 hyperthread
    type: Container
```

Close the file with control + d i.e. `^D`

```bash
# Create the CPU limit
k apply -f cpu-limit.yml
```

```
limitrange/cpu-limit-range created
```

```bash
# Create memory limit YAML file
cat > ~/memory-limit.yml
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - min:
      memory: 1Mi # Binary, 1 Mebibyte
    max:
      memory: 1024Mi # Binary, 1024 Mebibytes or 1 Gibibyte
    default:
      memory: 128Mi # Binary, 128 Mebibytes
    defaultRequest:
      memory: 64Mi # Binary, 64 Mebibytes
    type: Container
```

Close the file with control + d i.e. `^D`

```bash
# Create the CPU limit
k apply -f memory-limit.yml
```

```
limitrange/mem-limit-range created
```

```bash
# View the new limits.
k describe limitranges
```

```
Name:       cpu-limit-range
Namespace:  default
Type        Resource  Min  Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---  ---  ---------------  -------------  -----------------------
Container   cpu       10m  1    500m             1              -


Name:       mem-limit-range
Namespace:  default
Type        Resource  Min  Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---  ---  ---------------  -------------  -----------------------
Container   memory    1Mi  1Gi  64Mi             128Mi          -
```

```bash
# Create the Pod YAML.
cat > ~/pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: httpd-pod
  name: httpd-pod
spec:
  containers:
  - image: httpd:latest
    name: httpd-container
    resources:
      requests:
        cpu: 100m # 10% of 1 hyperthread
        memory: 15Mi # Binary, 15 Mebibytes.
      limits:
        cpu: 100m
        memory: 20Mi # Binary, 20 Mebibytes.
    ports:
    - containerPort: 80
```

Close the file with control + d i.e. `^D`

```bash
# Create the Pod
k apply -f pod.yml
```

```
pod/httpd-pod created
```

We are done.
