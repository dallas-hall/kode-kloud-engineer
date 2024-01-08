# httpd Password Protected Folder

## Task

> xFusionCorp Industries has hosted several static websites on Nautilus Application Servers in Stratos DC. There are some confidential directories in the document root that need to be password protected. Since they are using Apache for hosting the websites, the production support team has decided to use ``.htaccess` with basic auth. There is a website that needs to be uploaded to `/var/www/html/devops` on Nautilus App Server 2. However, we need to set up the authentication before that.
>
> * Create `/var/www/html/devops` direcotry if doesn't exist.
> * Add a user `mark` in `htpasswd`and set its password to `Rc5C9EyvbU`.
> * There is a file `/tmp/index.html` present on Jump Server. Copy the same to the directory you created, please make sure default document root should remain `/var/www/html`. Also website should work on URL `http://<app-server-hostname>:8080/devops/`

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* httpd basic auth packages
  * https://cwiki.apache.org/confluence/display/HTTPD/PasswordBasicAuth
  * https://www.arubacloud.com/tutorial/how-to-password-protect-directories-with-apache-on-centos-8.aspx

## Steps

```bash
# Copy over the html file from the jump box to the application servers.
scp /tmp/index.html steve@stapp02:/tmp

# Connect to application servers
ssh steve@stapp02

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check if Apache httpd is running, it wasn't.
systemctl status httpd

# Start and enable httpd
systemctl enable --now httpd

# Install networking tools, e.g. ss
dnf install -y iproute

# Find the httpd port, it was 8080.
ss -lntp | grep http
```

```
LISTEN 0      511          0.0.0.0:8080       0.0.0.0:*    users:(("httpd",pid=1220,fd=3),("httpd",pid=1219,fd=3),("httpd",pid=1218,fd=3),("httpd",pid=1192,fd=3))
```

```bash
# Testing connectivity worked.
curl -Lv http://localhost:8080/

# Test connectivity to devops didn't work, 404 error.
curl -Lv http://localhost:8080/devops
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /devops HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 404 Not Found
< Date: Mon, 08 Jan 2024 04:23:02 GMT
< Server: Apache/2.4.37 (centos)
< Content-Length: 196
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>
* Connection #0 to host localhost left intact
```

</details>

```bash
# Check the file system to see if the required path exists, it didn't.
ls -Ahl /var/www/html/

# Create the required path.
mkdir -p /var/www/html/devops

# Copy over the html file.
cp /tmp/index.html /var/www/html/devops/

# Test connectivity to devops worked. But the location isn't protected.
curl -Lv http://localhost:8080/devops
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /devops HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< Date: Mon, 08 Jan 2024 04:27:07 GMT
< Server: Apache/2.4.37 (centos)
< Location: http://localhost:8080/devops/
< Content-Length: 237
< Content-Type: text/html; charset=iso-8859-1
< 
* Ignoring the response-body
* Connection #0 to host localhost left intact
* Issue another request to this URL: 'http://localhost:8080/devops/'
* Found bundle for host localhost: 0x563f8ba71e50 [can pipeline]
* Re-using existing connection! (#0) with host localhost
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /devops/ HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Mon, 08 Jan 2024 04:27:07 GMT
< Server: Apache/2.4.37 (centos)
< Last-Modified: Mon, 08 Jan 2024 04:26:41 GMT
< ETag: "34-60e679ace53e5"
< Accept-Ranges: bytes
< Content-Length: 52
< Content-Type: text/html; charset=UTF-8
< 
This is xFusionCorp Industries Protected Directory!
* Connection #0 to host localhost left intact
```

</details>

```bash
# Create the httpd password file and the httpd user.
htpasswd -c /etc/httpd/.htpasswd mark
```

```
New password: 
Re-type new password: 
Adding password for user mark
```

```bash
# Create the appropriate configuration file.
cat >> /etc/httpd/conf.d/htpasswd.conf
```

```
<Directory "/var/www/html/devops">
  AuthType Basic
  AuthName "Authentication Required"
  AuthUserFile "/etc/httpd/.htpasswd"
  Require valid-user

  Order allow,deny
  Allow from all
</Directory>
```

Close the file with control + d i.e. `^D`

```bash
# Update 

# Reload httpd config and restart the service
systemctl reload httpd
systemctl restart httpd

# Testing connectivity worked.
curl -Lv http://localhost:8080/

# Test connectivity to devops without basic auth fails as expected.
curl -Lv http://localhost:8080/devops
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /devops HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 401 Unauthorized
< Date: Mon, 08 Jan 2024 04:39:31 GMT
< Server: Apache/2.4.37 (centos)
< WWW-Authenticate: Basic realm="Authentication Required"
< Content-Length: 381
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>401 Unauthorized</title>
</head><body>
<h1>Unauthorized</h1>
<p>This server could not verify that you
are authorized to access the document
requested.  Either you supplied the wrong
credentials (e.g., bad password), or your
browser doesn't understand how to supply
the credentials required.</p>
</body></html>
* Connection #0 to host localhost left intact
```

</details>

```bash
# Test connectivity to devops with basic auth works as expected.
curl -Lv --basic --user mark:Rc5C9EyvbU localhost:8080/devops
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8080 (#0)
* Server auth using Basic with user 'mark'
> GET /devops HTTP/1.1
> Host: localhost:8080
> Authorization: Basic bWFyazpSYzVDOUV5dmJV
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< Date: Mon, 08 Jan 2024 04:40:33 GMT
< Server: Apache/2.4.37 (centos)
< Location: http://localhost:8080/devops/
< Content-Length: 237
< Content-Type: text/html; charset=iso-8859-1
< 
* Ignoring the response-body
* Connection #0 to host localhost left intact
* Issue another request to this URL: 'http://localhost:8080/devops/'
* Found bundle for host localhost: 0x5605db1a4620 [can pipeline]
* Re-using existing connection! (#0) with host localhost
* Connected to localhost (127.0.0.1) port 8080 (#0)
* Server auth using Basic with user 'mark'
> GET /devops/ HTTP/1.1
> Host: localhost:8080
> Authorization: Basic bWFyazpSYzVDOUV5dmJV
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Mon, 08 Jan 2024 04:40:33 GMT
< Server: Apache/2.4.37 (centos)
< Last-Modified: Mon, 08 Jan 2024 04:26:41 GMT
< ETag: "34-60e679ace53e5"
< Accept-Ranges: bytes
< Content-Length: 52
< Content-Type: text/html; charset=UTF-8
< 
This is xFusionCorp Industries Protected Directory!
* Connection #0 to host localhost left intact
```

</details>

```bash
# Test connectivity from the jump host to devops with basic auth works as expected.
curl -Lv --basic --user mark:Rc5C9EyvbU stapp02:8080/devops
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
*   Trying 172.16.238.11...
* TCP_NODELAY set
* Connected to stapp02 (172.16.238.11) port 8080 (#0)
* Server auth using Basic with user 'mark'
> GET /devops HTTP/1.1
> Host: stapp02:8080
> Authorization: Basic bWFyazpSYzVDOUV5dmJV
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< Date: Mon, 08 Jan 2024 04:41:29 GMT
< Server: Apache/2.4.37 (centos)
< Location: http://stapp02:8080/devops/
< Content-Length: 235
< Content-Type: text/html; charset=iso-8859-1
< 
* Ignoring the response-body
* Connection #0 to host stapp02 left intact
* Issue another request to this URL: 'http://stapp02:8080/devops/'
* Found bundle for host stapp02: 0x564456ea1620 [can pipeline]
* Re-using existing connection! (#0) with host stapp02
* Connected to stapp02 (172.16.238.11) port 8080 (#0)
* Server auth using Basic with user 'mark'
> GET /devops/ HTTP/1.1
> Host: stapp02:8080
> Authorization: Basic bWFyazpSYzVDOUV5dmJV
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Mon, 08 Jan 2024 04:41:29 GMT
< Server: Apache/2.4.37 (centos)
< Last-Modified: Mon, 08 Jan 2024 04:26:41 GMT
< ETag: "34-60e679ace53e5"
< Accept-Ranges: bytes
< Content-Length: 52
< Content-Type: text/html; charset=UTF-8
< 
This is xFusionCorp Industries Protected Directory!
* Connection #0 to host stapp02 left intact
```

</details>

We are done.
