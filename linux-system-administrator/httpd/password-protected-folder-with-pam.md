# httpd Password Protected Folder

## Task

> We have a requirement where we want to password protect a directory in the Apache web server document root. We want to password protect `http://<website-url>:<apache_port>/protected` URL as per the following requirements (you can use any website-url for it like `localhost` since there are no such specific requirements as of now). Setup the same on App server 3 as per below mentioned requirements:<br><br>a. We want to use basic authentication.<br>b. We do not want to use `htpasswd` file based authentication. Instead, we want to use PAM authentication, i.e Basic Auth + PAM so that we can authenticate with a Linux user.<br>c. We already have a user `rose` with password `BruCStnMT5` which you need to provide access to.<br>d. You can access the website using APP button on the top bar.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* httpd basic auth packages - https://cwiki.apache.org/confluence/display/HTTPD/PasswordBasicAuth & https://www.server-world.info/en/note?os=CentOS_7&p=httpd&f=10

## Steps

```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Check if Apache httpd is running
systemctl status httpd
```

```
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd(8)
           man:apachectl(8)
```

```bash
# Start and enable httpd
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

```bash
# Find the httpd port
ss -lntp | grep http
```

```
LISTEN     0      511          *:8080                     *:*                   users:(("httpd",pid=725,fd=3),("httpd",pid=724,fd=3),("httpd",pid=723,fd=3),("httpd",pid=722,fd=3),("httpd",pid=721,fd=3),("httpd",pid=719,fd=3))
```

```bash
# Test connectivity
curl -Lv http://localhost:8080/
```

```
* About to connect() to localhost port 8080 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Tue, 11 Oct 2022 11:10:55 GMT
< Server: Apache/2.4.6 (CentOS) PHP/5.4.16
< Last-Modified: Thu, 30 Nov 2017 23:11:00 GMT
< ETag: "1a4-55f3b5d810900"
< Accept-Ranges: bytes
< Content-Length: 420
< Content-Type: text/html; charset=UTF-8
<
<?php
/**
 * Front to the WordPress application. This file doesn't do anything, but loads
 * wp-blog-header.php which does and tells WordPress to load the theme.
 *
 * @package WordPress
 */

/**
 * Tells WordPress to load the WordPress theme and output it.
 *
 * @var bool
 */
define( 'WP_USE_THEMES', true );

/** Loads the WordPress Environment and Template */
require( dirname( __FILE__ ) . '/wp-blog-header.php' );
* Connection #0 to host localhost left intact
```

```bash
# Test connectivity to protected
curl -Lv http://localhost:8080/protected
```

```
* About to connect() to localhost port 8080 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /protected HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
>
< HTTP/1.1 301 Moved Permanently
< Date: Tue, 11 Oct 2022 11:06:55 GMT
< Server: Apache/2.4.6 (CentOS) PHP/5.4.16
< Location: http://localhost:8080/protected/
< Content-Length: 240
< Content-Type: text/html; charset=iso-8859-1
<
* Ignoring the response-body
* Connection #0 to host localhost left intact
* Issue another request to this URL: 'http://localhost:8080/protected/'
* Found bundle for host localhost: 0x1436e90
* Re-using existing connection! (#0) with host localhost
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /protected/ HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Tue, 11 Oct 2022 11:06:55 GMT
< Server: Apache/2.4.6 (CentOS) PHP/5.4.16
< Last-Modified: Tue, 11 Oct 2022 11:00:58 GMT
< ETag: "26-5eac030e6700d"
< Accept-Ranges: bytes
< Content-Length: 38
< Content-Type: text/html; charset=UTF-8
<
This is KodeKloud Protected Directory
* Connection #0 to host localhost left intact
```

```bash
# Install the necessary packages
yum --enablerepo=epel -y install mod_authnz_external pwauth
```

```
...
Installed:
  mod_authnz_external.x86_64 0:3.3.3-3.el7      pwauth.x86_64 0:2.3.10-9.el7
...
```

```bash
# Update the appropriate configuration file.
cat >> /etc/httpd/conf.d/authnz_external.conf
```

```
<Directory /var/www/html/protected>
    #SSLRequireSSL # Commented out otherwise we need to install mod_ssl
    AuthType Basic
    AuthName "PAM Authentication"
    AuthBasicProvider external
    AuthExternal pwauth
    require valid-user
</Directory>
```

Close the file with control + d i.e. `^D`

```bash
# Reload httpd config and restart the service
systemctl reload httpd
systemctl restart httpd

# Test connectivity
curl -Lv http://localhost:8080/
```

```
* About to connect() to localhost port 8080 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Tue, 11 Oct 2022 11:10:55 GMT
< Server: Apache/2.4.6 (CentOS) PHP/5.4.16
< Last-Modified: Thu, 30 Nov 2017 23:11:00 GMT
< ETag: "1a4-55f3b5d810900"
< Accept-Ranges: bytes
< Content-Length: 420
< Content-Type: text/html; charset=UTF-8
<
<?php
/**
 * Front to the WordPress application. This file doesn't do anything, but loads
 * wp-blog-header.php which does and tells WordPress to load the theme.
 *
 * @package WordPress
 */

/**
 * Tells WordPress to load the WordPress theme and output it.
 *
 * @var bool
 */
define( 'WP_USE_THEMES', true );

/** Loads the WordPress Environment and Template */
require( dirname( __FILE__ ) . '/wp-blog-header.php' );
* Connection #0 to host localhost left intact
```

```bash
# Test connectivity to protected
curl -Lv http://localhost:8080/protected
```

```
* About to connect() to localhost port 8080 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /protected HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
>
< HTTP/1.1 401 Unauthorized
< Date: Tue, 11 Oct 2022 11:11:46 GMT
< Server: Apache/2.4.6 (CentOS) PHP/5.4.16
< WWW-Authenticate: Basic realm="PAM Authentication"
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

```bash
# Test connectivity to protected
curl -Lv --basic --user rose:BruCStnMT5 localhost:8080/protected
```

```
* About to connect() to localhost port 8080 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
* Server auth using Basic with user 'rose'
> GET /protected HTTP/1.1
> Authorization: Basic cm9zZTpCcnVDU3RuTVQ1
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
>
< HTTP/1.1 301 Moved Permanently
< Date: Tue, 11 Oct 2022 11:13:28 GMT
< Server: Apache/2.4.6 (CentOS) PHP/5.4.16
< Location: http://localhost:8080/protected/
< Content-Length: 240
< Content-Type: text/html; charset=iso-8859-1
<
* Ignoring the response-body
* Connection #0 to host localhost left intact
* Issue another request to this URL: 'http://localhost:8080/protected/'
* Found bundle for host localhost: 0x985f20
* Re-using existing connection! (#0) with host localhost
* Connected to localhost (127.0.0.1) port 8080 (#0)
* Server auth using Basic with user 'rose'
> GET /protected/ HTTP/1.1
> Authorization: Basic cm9zZTpCcnVDU3RuTVQ1
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Tue, 11 Oct 2022 11:13:28 GMT
< Server: Apache/2.4.6 (CentOS) PHP/5.4.16
< Last-Modified: Tue, 11 Oct 2022 11:00:58 GMT
< ETag: "26-5eac030e6700d"
< Accept-Ranges: bytes
< Content-Length: 38
< Content-Type: text/html; charset=UTF-8
<
This is KodeKloud Protected Directory
* Connection #0 to host localhost left intact
```
