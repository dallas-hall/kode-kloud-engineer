# Deploy LEMP Stack

## Task

> The Nautilus DevOps team want to deploy a static website on Kubernetes cluster. They are going to use Nginx, phpfpm and MySQL for the database. The team had already gathered the requirements and now they want to make this website live. Below you can find more details:
>
> * Create some secrets for MySQL.
>   * Create a secret named `mysql-root-pass` wih key/value pairs `name: password` and `value: R00t`
>   * Create a secret named `mysql-user-pass` with key/value pairs `name: username` and `value: kodekloud_gem` and `name: password` and `value: Rc5C9EyvbU`
>   * Create a secret named `mysql-db-url` with key/value pairs `name: database` and `value: kodekloud_db6`
>   * Create a secret named `mysql-host` with key/value pairs `name: host` and `value: mysql-service`
> * Create a config map `php-config` for `php.ini` with `variables_order = "EGPCS"` data.
> * Create a deployment named `lemp-wp`.
> * Create two containers under it. First container must be `nginx-php-container` using image `webdevops/php-nginx:alpine-3-php7` and second container must be `mysql-container` from `image mysql:5.6`. Mount `php-config` configmap in nginx container at `/opt/docker/etc/php/php.ini` location.
> * Add some environment variables for both containers:
>   * `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD` and `MYSQL_HOST`. Take their values from the secrets you created. Please make sure to use `env` field (do not use envFrom) to define the name-value pair of environment variables.
> * Create a node port type service `lemp-service` to expose the web application, `nodePort` must be `30008`.
> * Create a service for mysql named `mysql-service` and its port must be `3306`.
> * We already have a `/tmp/index.php` file on jump_host server. Copy this file into the nginx container under document root i.e `/app` and replace the dummy values for mysql related variables with the environment variables you have set for mysql related parameters. Please make sure you do not hard code the mysql related details in this file, you must use the environment variables to fetch those values.
> Once done, you must be able to access this website using Website button on the top bar, please note that you should see `Connected successfully` message while accessing this page.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* What is LEMP (Linux, nginx, MySQL, PHP).
  * https://www.digitalocean.com/community/tutorials/what-is-lemp

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   66m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox. Click the App button gives an nginx 502 bad gate.

```bash
# Create the Secrets
k create secret generic mysql-root-pass --from-literal password=R00t
k create secret generic mysql-user-pass --from-literal username=kodekloud_gem --from-literal password=Rc5C9EyvbU
k create secret generic mysql-db-url --from-literal database=kodekloud_db6
k create secret generic mysql-host --from-literal host=mysql-service

# Create the ConfigMap
cat > cm.yaml
```


```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: php-config
data:
  php.ini: |
    variables_order = "EGPCS"
```

This is equal to the below:

```bash
echo 'variables_order = "EGPCS"' > /tmp/php.ini
k create cm php-config-2 --from-file /tmp/php.ini

# Create the Deployment.
cat > d.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: lemp-wp
  name: lemp-wp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lemp-wp
  template:
    metadata:
      labels:
        app: lemp-wp
    spec:
      containers:
      - image: mysql:5.6
        name: mysql-container
        ports:
        - containerPort: 3306
        env: &env_var # Create a Go Anchor (i.e. macro)
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-pass
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-db-url
              key: database
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: password
        - name: MYSQL_HOST
          valueFrom:
            secretKeyRef:
              name: mysql-host
              key: host
      - image: webdevops/php-nginx:alpine-3-php7
        name: nginx-php-container
        ports:
        - containerPort: 80
        volumeMounts:
        - name: php-ini
          mountPath: /opt/docker/etc/php/
        env: *env_var # Reference a Go Anchor (i.e. macro). This will inject everything inside of of the Anchor here.
      volumes:
      - name: php-ini
        configMap:
          name: php-config
          items:
          - key: php.ini
            path: php.ini

```

```bash
# Create the Deployment.
k apply -f d.yaml

# Create ClusterIP Service.
k expose deployment lemp-wp --port=3306 --name=mysql-service

# Create NodePort Service YAML.
cat > np.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: lemp-wp
  name: lemp-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30008
  selector:
    app: lemp-wp
  type: NodePort
```

Close the file with control + d i.e. `^D`

```bash
# Create the NodePort Service.
k apply -f np.yaml

# Update the index.php file.
cp -a /tmp/index.php /tmp/index.php.old
cat > /tmp/index.php
```

```php
<?php
$dbname = getenv('MYSQL_DATABASE');
$dbuser = getenv('MYSQL_USER');
$dbpass = getenv('MYSQL_PASSWORD');
$dbhost = getenv('MYSQL_HOST');

$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($test_query);

if ($result->connect_error) {
   die("Connection failed: " . $conn->connect_error);
}
  echo "Connected successfully";
?>
```

```bash
# Get the Pod name
k get po

# Copy the index.php file into the container.
k cp /tmp/index.php lemp-wp-6cc679dcf7-wh677:/app/index.php -c nginx-php-container

# The connection
curl kodekloud-control-plane:30008
```

```
Connected successfully
```

Testing by clicking the App buttons also shows it worked. We are done.

Kode Kloud Engineer review suggestion:

> I think the problem is with php-ini volumeMount in the nginx-php-container in your deployment yaml file. The mountPath should be `/opt/docker/etc/php` (omitting the final `php.ini`). This is because the path at which the volume is mounted is actually the concatenation of mountPath and subPath (the location of the volume would be `{mountPath}/{subPath}` or in your case `{/opt/docker/etc/php}/{php.ini}`)