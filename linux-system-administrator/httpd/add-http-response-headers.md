# Add HTTP Response Header

## Task

> We are working on hardening Apache web server on all app servers. As a part of this process we want to add some of the Apache response headers for security purpose. We are testing the settings one by one on all app servers. As per details mentioned below enable these headers for Apache:<br><br>Install `httpd` package on App Server 1 using `yum` and configure it to run on 6300 port, make sure to start its service.<br><br>Create an index.html file under Apache's default document root i.e `/var/www/html` and add below given content in it.<br><br>Welcome to the xFusionCorp Industries!<br><br>Configure Apache to enable below mentioned headers:<br>`X-XSS-Protection` header with value `1; mode=block`<br>`X-Frame-Options` header with value `SAMEORIGIN`<br>`X-Content-Type-Options` header with value `nosniff`<br><br>**Note:** You can test using `curl` on the given app server as LBR URL will not work for this task.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Change listening port - https://httpd.apache.org/docs/2.4/en/bind.html
* Add response header - https://stackoverflow.com/questions/35564735/how-do-servers-set-http-response-headers & https://httpd.apache.org/docs/2.4/mod/mod_headers.html

## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Install httpd
yum install -y httpd
```

```
...
Installed:
  httpd.x86_64 0:2.4.6-97.el7.centos.5
...
```

```bash
# Check which port https is listening on
grep -i listen /etc/httpd/conf/httpd.conf
```

```
...
Listen 80
```

```bash
# Take a backup
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak

# Change the port httpd is listening on
sed -i -r 's/Listen 80/Listen 6300/g' /etc/httpd/conf/httpd.conf

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
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

```bash
# Check httpd
curl -v localhost:6300
```

```
* About to connect() to localhost port 6300 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 6300 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:6300
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Sat, 10 Sep 2022 23:37:24 GMT
< Server: Apache/2.4.6 (CentOS)
< Last-Modified: Sat, 10 Sep 2022 23:29:52 GMT
< ETag: "27-5e85b0a1d068d"
< Accept-Ranges: bytes
< Content-Length: 39
< Content-Type: text/html; charset=UTF-8
<
Welcome to the xFusionCorp Industries!
* Connection #0 to host localhost left intact
```

```bash
# Add the http response headers at the end of the file
cat >> /etc/httpd/conf/httpd.conf
```

```
# Adding custom headers
#
Header set X-XSS-Protection "1; mode=block"
Header set X-Frame-Options "SAMEORIGIN"
Header set X-Content-Type-Options "nosniff"
```

Close the file with control + d i.e. `^D`

```bash
# Reload httpd to pick up the new configuration
systemctl reload httpd

# Test changes
curl -v localhost:6300
```

```
* About to connect() to localhost port 6300 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 6300 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:6300
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Sat, 10 Sep 2022 23:44:19 GMT
< Server: Apache/2.4.6 (CentOS)
< Last-Modified: Sat, 10 Sep 2022 23:29:52 GMT
< ETag: "27-5e85b0a1d068d"
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

We can see the response headers in the verbose output.

```bash
# Test httpd from thor jump box
curl -v stapp01:6300
```

```
* About to connect() to stapp01 port 6300 (#0)
*   Trying 172.16.238.10...
* Connected to stapp01 (172.16.238.10) port 6300 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: stapp01:6300
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Sat, 10 Sep 2022 23:45:05 GMT
< Server: Apache/2.4.6 (CentOS)
< Last-Modified: Sat, 10 Sep 2022 23:29:52 GMT
< ETag: "27-5e85b0a1d068d"
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

We can see the response headers in the verbose output.
