# phpfpm Deployment

## Task

> The Nautilus Application Development team is planning to deploy one of the php-based application on Kubernetes cluster. As per discussion with DevOps team they have decided to use `nginx` and `phpfpm`. Additionally, they shared some custom configuration requirements. Below you can find more details. Please complete the task as per requirements mentioned below:
> 1. Create a service to expose this app, the service type must be `NodePort`, nodePort should be `30012`.
> 2. Create a config map `nginx-config` for `nginx.conf` as we want to add some custom settings for `nginx.conf`.
>     * Change default port `80` to `8095` in nginx.conf.
>     * Change default document root `/usr/share/nginx` to `/var/www/html` in `nginx.conf`.
>     * Update directory index to `index index.html index.htm index.php` in `nginx.conf`.
> 3. Create a pod named `nginx-phpfpm` .
>     * Create a shared volume `shared-files` that will be used by both containers (nginx and phpfpm) also it should be a `emptyDir` volume.
>     * Map the `ConfigMap` we declared above as a volume for nginx container. Name the volume as `nginx-config-volume`, mount path should be `/etc/nginx/nginx.conf` and `subPath` should be `nginx.conf`.
>     * Nginx container should be named as `nginx-container` and it should use `nginx:latest` image. PhpFPM container should be named as `php-fpm-container` and it should use `php:8.2-fpm-alpine` image.
>     * The shared volume `shared-files` should be mounted at `/var/www/html` location in both containers. Copy `/opt/index.php` from jump host to the nginx document root inside `nginx` container, once done you can access the app using App

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* K8s emptyDir Volume
  * https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
  * https://kubernetes.io/docs/concepts/storage/volumes/#configmap
* Config Map
  * https://stackoverflow.com/a/64179010
* NodePort
  * https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   46m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

```bash
# Create Config Map YAML
cat > cm.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    user nginx;
    worker_processes  1;
    events {
      worker_connections  10240;
    }
    http {
      server {
        listen 8095;
        server_name localhost;
        location / {
          root /var/www/html;
          index index.html index.htm index.php;
        }
      }
    }
```

Close the file with control + d i.e. `^D`

```bash
# Create Config Map
k apply -f cm.yaml

# Create Pod YAML
cat > pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
  labels:
    app: nginx-phpfpm
spec:
  containers:
  - image: nginx:latest
    name: nginx-container
    ports:
    - containerPort: 8095
    volumeMounts:
    - mountPath: /var/www/html
      name: shared-files
    - mountPath: /etc/nginx/nginx.conf
      name: nginx-config-volume
      subPath: nginx.conf
  - image: php:8.2-fpm-alpine
    name: php-fpm-container
    ports:
    - containerPort: 8095
    volumeMounts:
    - mountPath: /var/www/html
      name: shared-files
  volumes:
  - name: shared-files
    emptyDir:
      sizeLimit: 128Mi
  - name: nginx-config-volume
    configMap:
      name: nginx-config
      items:
      - key: nginx.conf
        path: nginx.conf
```

Close the file with control + d i.e. `^D`


```bash
# Create Pod
k apply -f pod.yaml

# Create NodePort YAML
cat > svc.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx-phpfpm
  name: nginx-phpfpm
spec:
  ports:
  - port: 8095
    protocol: TCP
    targetPort: 8095
    nodePort: 30012
  selector:
    app: nginx-phpfpm
  type: NodePort
status:
  loadBalancer: {}
```

Close the file with control + d i.e. `^D`

```bash
# Create NodePort
k apply -f svc.yaml

# Get homepage content
cat /opt/index.php
```

```
It works!
```

```bash
# Create homepage
k exec nginx-phpfpm -c php-fpm-container -- sh -c 'echo "It works!" > /var/www/html/index.php'
k exec nginx-phpfpm -c php-fpm-container -- cat /var/www/html/index.php
```

```
It works!
```

Testing by clicking the App buttons shows it worked. We are done.
