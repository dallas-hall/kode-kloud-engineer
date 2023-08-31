# Create A Job

## Task

>The Nautilus DevOps team is working on to create few jobs in Kubernetes cluster. They might come up with some real scripts/commands to use, but for now they are preparing the templates and testing the jobs with dummy commands. Please create a job template as per details given below:
>
>Create a job `countdown-datacenter`.
>
>The spec template should be named as `countdown-datacenter` (under metadata), and the container should be named as `container-countdown-datacenter`
>
>Use image ubuntu with latest tag only and remember to mention tag i.e `debian:latest`, and restart policy should be Never.
>
>Use `command sleep 5`

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Jobs
  * https://kubernetes.io/docs/concepts/workloads/controllers/job/
  * https://kubernetes.io/docs/tasks/job/job-with-pod-to-pod-communication/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   10m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Create Job YAML
cat > job.yaml
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: countdown-datacenter
spec:
  template:
    metadata:
      name: countdown-datacenter
    spec:
      containers:
      - name: container-countdown-datacenter
        image: debian:latest
        command:
        - /bin/sh
        - -c
        - sleep 5
      restartPolicy: Never
```

Close the file with control + d i.e. `^D`


```bash
# Create the Job
k apply -f  job.yaml
```

```
job.batch/countdown-datacenter created
```

```bash
# Check the Job
k get job
```

```
NAME                 COMPLETIONS   DURATION   AGE
countdown-datacenter   1/1           15s        23s
```

We are done.
