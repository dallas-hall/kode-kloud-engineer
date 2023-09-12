# Volume Mount Troubleshooting

## Task

> We deployed a Nginx and PHP-FPM based setup on Kubernetes cluster last week and it had been working fine so far. This morning one of the team members made a change somewhere which caused some issues, and it stopped working. Please look into the issue and fix it:
>
> The pod name is `nginx-phpfpm` and configmap name is `nginx-config`. Figure out the issue and fix the same.
>
> Once issue is fixed, copy `/home/thor/index.php` file from the jump host to the `nginx-container` under nginx document root and you should be able to access the website using Website button on top bar.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* Copy to and from Pods
  * https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   28m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Check the ConfigMap
k describe cm
```

```
Name:         nginx-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
nginx.conf:
----
events {
}
http {
  server {
    listen 8099 default_server;
    listen [::]:8099 default_server;

    # Set nginx to serve files from the shared volume!
    root /var/www/html;
    index  index.html index.htm index.php;
    server_name _;
    location / {
      try_files $uri $uri/ =404;
    }
    location ~ \.php$ {
      include fastcgi_params;
      fastcgi_param REQUEST_METHOD $request_method;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_pass 127.0.0.1:9000;
    }
  }
}


BinaryData
====

Events:  <none>
```

```bash
# Check the Service
k get svc
```

```
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP          30m
nginx-service   NodePort    10.96.180.23   <none>        8099:30008/TCP   9m48s
```

Looks okay as the ports match, need to dig deeper.

```bash
# Check the Pod
k get po nginx-phpfpm
```

```
NAME           READY   STATUS    RESTARTS   AGE
nginx-phpfpm   2/2     Running   0          8m38s
```

Looks okay as the Pods are running, need to dig deeper.

```bash
# Test connectivity.
curl kodekloud-control-plane:30008
```

```html
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.25.2</center>
</body>
</html>
```

The homepage at `/var/www/html` is missing.

```bash
# Check the Pod some more.
k describe po nginx-phpfpm
```

```
Name:             nginx-phpfpm
Namespace:        default
Priority:         0
Service Account:  default
Node:             kodekloud-control-plane/172.17.0.2
Start Time:       Tue, 12 Sep 2023 08:21:13 +0000
Labels:           app=php-app
Annotations:      <none>
Status:           Running
IP:               10.244.0.5
IPs:
  IP:  10.244.0.5
Containers:
  php-fpm-container:
    Container ID:   containerd://27a37ad0be859b7362117113786d3ef40b6e502fd3d9d04ceb4c575bd8df4184
    Image:          php:7.2-fpm-alpine
    Image ID:       docker.io/library/php@sha256:2e2d92415f3fc552e9a62548d1235f852c864fcdc94bcf2905805d92baefc87f
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 12 Sep 2023 08:21:20 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8jsbf (ro)
      /var/www/html from shared-files (rw)
  nginx-container:
    Container ID:   containerd://94961fad299466aab6c6d6a8d31565eb79d187510b8f7599ae18701c87e3c264
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:6926dd802f40e5e7257fded83e0d8030039642e4e10c4a98a6478e9c6fe06153
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 12 Sep 2023 08:21:30 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /etc/nginx/nginx.conf from nginx-config-volume (rw,path="nginx.conf")
      /usr/share/nginx/html from shared-files (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8jsbf (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  shared-files:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  nginx-config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      nginx-config
    Optional:  false
  kube-api-access-8jsbf:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  9m5s   default-scheduler  Successfully assigned default/nginx-phpfpm to kodekloud-control-plane
  Normal  Pulling    9m5s   kubelet            Pulling image "php:7.2-fpm-alpine"
  Normal  Pulled     8m59s  kubelet            Successfully pulled image "php:7.2-fpm-alpine" in 6.026029669s (6.026049922s including waiting)
  Normal  Created    8m59s  kubelet            Created container php-fpm-container
  Normal  Started    8m58s  kubelet            Started container php-fpm-container
  Normal  Pulling    8m58s  kubelet            Pulling image "nginx:latest"
  Normal  Pulled     8m48s  kubelet            Successfully pulled image "nginx:latest" in 10.144247478s (10.144262804s including waiting)
  Normal  Created    8m48s  kubelet            Created container nginx-container
  Normal  Started    8m48s  kubelet            Started container nginx-container
```

The `nginx-container` is missing the `/var/www/html` mount that was seen in the ConfigMap.

```bash
# Backup the Pod YAML
k get po nginx-phpfpm -o yaml > pod.yaml

# Update the YAML
k edit po nginx-phpfpm
```

Update `/usr/share/nginx/html` to `/var/www/html` and save the file with `:x` but the update will error.

```
error: pods "nginx-phpfpm" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-1222739582.yaml"
error: Edit cancelled, no valid changes were saved.
```

```bash
# Delete the Pod since it wasn't updated.
k delete po nginx-phpfpm
```

```
pod "nginx-phpfpm" deleted
```

```bash
# Receate the Pod from the temporary file.
k apply -f /tmp/kubectl-edit-1222739582.yaml
```

```
pod/nginx-phpfpm created
```

```bash
# Copy the file over.
k cp /home/thor/index.php nginx-phpfpm:/var/www/html -c nginx-container

# Test connectivity.
curl kodekloud-control-plane:30008
```

```
Shit load of HTML from php_info() here. :)
```

We are done.
