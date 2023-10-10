# Add HTTP Response Header

## Task

> We are working on hardening Apache web server on all app servers. As a part of this process we want to add some of the Apache response headers for security purpose. We are testing the settings one by one on all app servers. As per details mentioned below enable these headers for Apache:
>
> * Install `httpd` package on *App Server 1* using `yum` and configure it to run on `8088` port, make sure to start its service.
> * Create an `index.html` file under Apache's default document root i.e `/var/www/html` and add below given content in it.
>
> `Welcome to the xFusionCorp Industries!`
>
> * Configure Apache to enable below mentioned headers:
>   * `X-XSS-Protection` header with value `1; mode=block`
>   * `X-Frame-Options` header with value `SAMEORIGIN`
>   * `X-Content-Type-Options` header with value `nosniff`
>
> **Note:** You can test using curl on the given app server as LBR URL will not work for this task.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Change listening port
  * https://httpd.apache.org/docs/2.4/en/bind.html
* Add response header
  * https://stackoverflow.com/questions/35564735/how-do-servers-set-http-response-headers
  * https://httpd.apache.org/docs/2.4/mod/mod_headers.html

## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Install httpd
yum install -y httpd

# Check which port https is listening on. It was the default port 80.
grep -i listen /etc/httpd/conf/httpd.conf

# Take a backup
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.old

# Change the port httpd is listening on
sed -i -r 's/Listen 80/Listen 8088/g' /etc/httpd/conf/httpd.conf

# Create the landing page
cat > /var/www/html/index.html
```

```
Welcome to the xFusionCorp Industries!
```

Close the file with control + d i.e. `^D`

```bash
# Start and enable the httpd service
systemctl enable --now httpd

# Check httpd
curl -v localhost:8088
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* Rebuilt URL to: localhost:8088/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8088 (#0)
> GET / HTTP/1.1
> Host: localhost:8088
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Tue, 10 Oct 2023 08:30:26 GMT
< Server: Apache/2.4.37 (CentOS Stream)
< Last-Modified: Tue, 10 Oct 2023 08:30:01 GMT
< ETag: "27-60758834827bb"
< Accept-Ranges: bytes
< Content-Length: 39
< Content-Type: text/html; charset=UTF-8
<
Welcome to the xFusionCorp Industries!
* Connection #0 to host localhost left intact
```

</details>

```bash
# Add the http response headers at the end of the file
cat >> /etc/httpd/conf/httpd.conf
```

```
# Adding custom headers
Header set X-XSS-Protection "1; mode=block"
Header set X-Frame-Options "SAMEORIGIN"
Header set X-Content-Type-Options "nosniff"
```

Close the file with control + d i.e. `^D`

```bash
# Reload httpd to pick up the new configuration
systemctl reload httpd

# Test changes
curl -v localhost:8088
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
# Adding custom headers
Header set X-XSS-Protection "1; mode=block"
Header set X-Frame-Options "SAMEORIGIN"
Header set X-Content-Type-Options "nosniff"
[root@stapp01 ~]# systemctl reload httpd
[root@stapp01 ~]# curl -v localhost:8088
* Rebuilt URL to: localhost:8088/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8088 (#0)
> GET / HTTP/1.1
> Host: localhost:8088
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Tue, 10 Oct 2023 08:31:30 GMT
< Server: Apache/2.4.37 (CentOS Stream)
< Last-Modified: Tue, 10 Oct 2023 08:30:01 GMT
< ETag: "27-60758834827bb"
< Accept-Ranges: bytes
< Content-Length: 39
< X-XSS-Protection: 1; mode=block
< X-Frame-Options: SAMEORIGIN
< X-Content-Type-Options: nosniff
< Content-Type: text/html; charset=UTF-8
<
Welcome to the xFusionCorp Industries!
* Connection #0 to host localhost left intact
```

</details>

We can see the response headers in the verbose output.

```bash
# Test httpd from thor jump box
curl -v stapp01:8088
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* Rebuilt URL to: stapp01:8088/
*   Trying 172.16.238.10...
* TCP_NODELAY set
* Connected to stapp01 (172.16.238.10) port 8088 (#0)
> GET / HTTP/1.1
> Host: stapp01:8088
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Tue, 10 Oct 2023 08:32:27 GMT
< Server: Apache/2.4.37 (CentOS Stream)
< Last-Modified: Tue, 10 Oct 2023 08:30:01 GMT
< ETag: "27-60758834827bb"
< Accept-Ranges: bytes
< Content-Length: 39
< X-XSS-Protection: 1; mode=block
< X-Frame-Options: SAMEORIGIN
< X-Content-Type-Options: nosniff
< Content-Type: text/html; charset=UTF-8
<
Welcome to the xFusionCorp Industries!
* Connection #0 to host stapp01 left intact
```

</details>

We can see the same response headers as above, we are done.
