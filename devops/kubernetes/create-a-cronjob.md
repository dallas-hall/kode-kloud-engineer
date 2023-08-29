# Create A CronJob

## Task

> There are some jobs/tasks that need to be run regularly on different schedules. Currently the Nautilus DevOps team is working on developing some scripts that will be executed on different schedules, but for the time being the team is creating some cron jobs in Kubernetes cluster with some dummy commands (which will be replaced by original scripts later). Create a cronjob as per details given below:
>
> * Create a cronjob named `xfusion`.
> * Set Its schedule to something like `*/4 * * * *`, you set any schedule for now
> * Container name should be `cron-xfusion`.<br>Use httpd image with latest tag only and remember to mention the tag i.e `httpd:latest`.
> * Run a dummy command `echo Welcome to xfusioncorp!`.<br>
> * Ensure restart policy is `OnFailure`.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* CronJobs
  * https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
  * https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   16m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Create Pod YAML
cat > cronjob.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: xfusion
spec:
  schedule: "*/4 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cron-xfusion
            image: httpd:latest
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - echo Welcome to xfusioncorp!
          restartPolicy: OnFailure
```

Close the file with control + d i.e. `^D`


```bash
# Create the Pod
k apply -f  cronjob.yaml
```

```
kubecpod/pod-httpd created
```

```bash
# Check the Pod
k get cj
```

```
NAME      SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
xfusion   */4 * * * *   False     0        <none>          55s
```

We are done.
