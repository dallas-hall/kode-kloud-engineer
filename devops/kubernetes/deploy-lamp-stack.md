# LAMP Stack Deployment

## Task

> The Nautilus DevOps team want to deploy a PHP website on Kubernetes cluster. They are going to use Apache as a web server and Mysql for database. The team had already gathered the requirements and now they want to make this website live. Below you can find more details:
>
> * Create a config map `php-config` for `php.ini` with `variables_order = "EGPCS"` data.
> * Create a deployment named `lamp-wp`.
> * Create two containers under it. First container must be `httpd-php-container` using image `webdevops/php-apache:alpine-3-php7` and second container must be `mysql-container` from image `mysql:5.6`. Mount `php-config` configmap in httpd container at `/opt/docker/etc/php/php.ini` location.
> * Create kubernetes generic secrets for mysql related values like myql root password, mysql user, mysql password, mysql host and mysql database. Set any values of your choice.
> * Add some environment variables for both containers:
>   * `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD` and `MYSQL_HOST`. Take their values from the secrets you created. Please make sure to use env field (**do not use envFrom**) to define the name-value pair of environment variables.
> * Create a node port type service `lamp-service` to expose the web application, nodePort must be `30008`.
> * Create a service for mysql named `mysql-service` and its port must be `3306`.
> * We already have `/tmp/index.php` file on jump_host server.
>   * Copy this file into httpd container under Apache document root i.e `/app` and replace the dummy values for mysql related variables with the environment variables you have set for mysql related parameters. Please make sure you do not hard code the mysql related details in this file, you must use the environment variables to fetch those values.
>   * You must be able to access this `index.php` on node port `30008` at the end, please note that you should see `Connected successfully` message while accessing this page.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* ConfigMap
  * https://kubernetes.io/docs/concepts/configuration/configmap/
  * https://stackoverflow.com/questions/61138081/how-to-add-php-ini-to-a-configmap-in-kubernetes-yaml
* ConfigMap as Volume.
  * https://kubernetes.io/docs/concepts/storage/volumes/#configmap
* Secrets.
  * https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data
* Volumes
  * https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath
* PHP environment variables
  * https://www.php.net/manual/en/reserved.variables.environment.php
  * https://www.php.net/manual/en/function.getenv.php
* `mysqli_connect`
  * https://www.w3schools.com/php/func_mysqli_connect.asp
* Some advice from https://kodekloud.com/community/t/deploy-lamp-stack-on-kubernetes-cluster-configmap-mount-error/362394/18

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES           AGE   VERSION                     INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                      KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane   20m   v1.27.3-44+b5c876a05b7bbd   172.17.0.2    <none>        Ubuntu Mantic Minotaur (development branch)   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

We can connect fine from Thor jumpbox.

```bash
# Create the ConfigMap YAML
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

Close the file with control + d i.e. `^D`


```bash
k apply -f cm.yaml

# Create Secrets.
k create secret generic mysql --from-literal=MYSQL_ROOT_PASSWORD='Abc123!@#' --from-literal=MYSQL_DATABASE=php --from-literal=MYSQL_USER=php-app --from-literal=MYSQL_PASSWORD='Abc123!@#' --from-literal=MYSQL_HOST=mysql-service

# Create Deployment YAML file.
cat > deploy.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: lamp-wp
  name: lamp-wp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lamp-wp
  template:
    metadata:
      labels:
        app: lamp-wp
    spec:
      containers:
      - image: mysql:5.6
        name: mysql-container
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql
              key: MYSQL_PASSWORD
        - name: MYSQL_HOST
          valueFrom:
            secretKeyRef:
              name: mysql
              key: MYSQL_HOST
      - image: webdevops/php-apache:alpine-3-php7
        name: httpd-php-container
        ports:
        - containerPort: 80
        volumeMounts:
        - name: php-ini
          mountPath: /opt/docker/etc/php/php.ini
          subPath: php.ini
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql
              key: MYSQL_PASSWORD
        - name: MYSQL_HOST
          valueFrom:
            secretKeyRef:
              name: mysql
              key: MYSQL_HOST
      volumes:
      - name: php-ini
        configMap:
          name: php-config
          items:
          - key: php.ini
            path: php.ini
```

Close the file with control + d i.e. `^D`

```bash
# Create the Deployment.
k apply -f deploy.yaml

# Create ClusterIP Service.
k expose deployment lamp-wp --port=3306 --name=mysql-service

# Create NodePort Service YAML.
cat > np.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: lamp-wp
  name: lamp-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30008
  selector:
    app: lamp-wp
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
k cp /tmp/index.php lamp-wp-67894f8497-fv6n2:/app/index.php -c httpd-php-container

# The connection
curl kodekloud-control-plane:30008
```

```
Connected successfully
```

Testing by clicking the App buttons also shows it worked. We are done.
