# LAMP Stack Deployment Troubleshooting

## Task

> One of the DevOps team member was trying to install a WordPress website on a LAMP stack which is essentially deployed on Kubernetes cluster. It was working well and we could see the installation page a few hours ago. However something is messed up with the stack now due to a website went down. Please look into the issue and fix it:
>
> FYI, deployment name is `lamp-wp` and its using a service named `lamp-service`. The Apache is using http default port and nodeport is `30008`. From the application logs it has been identified that application is facing some issues while connecting to the database in addition to other issues. Additionally, there are some environment variables associated with the pods like `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_HOST`.

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* PHP environment variables
  * https://www.php.net/manual/en/reserved.variables.environment.php
  * https://www.php.net/manual/en/function.getenv.php
* `mysqli_connect`
  * https://www.w3schools.com/php/func_mysqli_connect.asp

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

We can connect fine from Thor jumpbox. Click the App button gives an nginx 502 bad gate.

```bash
# Check the Deployment, the Pods were running.
k get deploy lamp-wp

# Check the Pod logs.
k logs lamp-wp-56c7c454fc-j4x9g
```

There were no glaring issues. Click the App button gives an nginx 502 bad gate.

```bash
# Check the service.
k get svc
```

The Pod port is 8080 instead of 80. Using `k edit svc lamp-service` change 8080 to 80 and press `:x` to save and quit.

Clicking the App button brings up a blank screen. Checking the Pod logs again shows this error:

```
[27-Nov-2023 23:37:07] WARNING: [pool www] child 222 said into stderr: "NOTICE: PHP message: PHP Notice:  Undefined index: MYSQL_PASSWORDS in /app/index.php on line 4"
[27-Nov-2023 23:37:07] WARNING: [pool www] child 222 said into stderr: "NOTICE: PHP message: PHP Notice:  Undefined index: HOST_MYQSL in /app/index.php on line 5"
[php-fpm:access] 127.0.0.1 -  27/Nov/2023:23:37:07 +0000 "GET /index.php" 200 /app/index.php 8.695 2048 0.00%
[27-Nov-2023 23:37:07] WARNING: [pool www] child 222 said into stderr: "NOTICE: PHP message: PHP Warning:  mysqli_connect(): (HY000/2005): Unknown MySQL server host 'mysql' (-2) in /app/index.php on line 8"
[Mon Nov 27 23:37:07.466487 2023] [proxy_fcgi:error] [pid 163:tid 140137383238320] [client 10.244.0.1:26552] AH01071: Got error 'PHP message: PHP Notice:  Undefined index: MYSQL_PASSWORDS in /app/index.php on line 4\nPHP message: PHP Notice:  Undefined index: HOST_MYQSL in /app/index.php on line 5\nPHP message: PHP Warning:  mysqli_connect(): (HY000/2005): Unknown MySQL server host 'mysql' (-2) in /app/index.php on line 8\n', referer: https://59cac5a54b544fe5.labs.kodekloud.com/
10.244.0.1 - - [27/Nov/2023:23:37:07 +0000] "GET / HTTP/1.1" 200 23 "https://59cac5a54b544fe5.labs.kodekloud.com/" "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0"
[httpd:access] 30008-port-59cac5a54b544fe5.labs.kodekloud.com:80 10.244.0.1 - - [27/Nov/2023:23:37:07 +0000] "GET / HTTP/1.1" 200 bytesIn:2026 bytesOut:207 reqTime:0
```

There is a 500 internal server error with the `index.php` file. Checking the container with `k exec lamp-wp-56c7c454fc-c2g4j -c httpd-php-container -- cat /app/index.php` shows

```php
<?php
$dbname = $_ENV['MYSQL_DATABASE'];
$dbuser = $_ENV['MYSQL_USER'];
$dbpass = $_ENV['MYSQL_PASSWORDS'];
$dbhost = $_ENV['HOST_MYQSL'];


$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($test_query);

if ($result->connect_error) {
   die("Connection failed: " . $conn->connect_error);
}
  echo "Connected successfully";
```

There is naming error for `MYSQL_PASSWORDS` and `HOST_MYQSL` and the file is missing a closing `?>`

```bash
# Fix the container
k exec lamp-wp-56c7c454fc-c2g4j -c httpd-php-container -it -- bash
cat >  /app/index.php
```

```php
<?php
$dbname = $_ENV['MYSQL_DATABASE'];
$dbuser = $_ENV['MYSQL_USER'];
$dbpass = $_ENV['MYSQL_PASSWORD'];
$dbhost = $_ENV['MYSQL_HOST'];


$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($test_query);

if ($result->connect_error) {
   die("Connection failed: " . $conn->connect_error);
}
  echo "Connected successfully";
?>
```

Close the file with control + d i.e. `^D`

Pressing the App button shows the connection was successful. We are done.
