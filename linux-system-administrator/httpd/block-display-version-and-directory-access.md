# Add HTTP Response Header

## Task

> During a recent security audit, the application security team of xFusionCorp Industries found security issues with the Apache web server on Nautilus App Server 1 server in Stratos DC. They have listed several security issues that need to be fixed on this server. Please apply the security settings below:<br><br>a. On Nautilus App Server 1 it was identified that the Apache web server is exposing the version number. Ensure this server has the appropriate settings to hide the version number of the Apache web server.<br><br>b. There is a website hosted under `/var/www/html/blog` on App Server 1. It was detected that the directory `/blog` lists all of its contents while browsing the URL. Disable the directory browser listing in Apache config.<br><br>c. Also make sure to restart the Apache service after making the changes.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Turn off version numbers - https://www.tecmint.com/hide-apache-web-server-version-information/
* Block access to a folder - https://www.tecmint.com/disable-apache-directory-listing-htaccess/

## Steps


```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Start and enable httpd if it already isn't.
systemctl enable --now httpd
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

It wasn't running so it got enabled and started.

```bash
# View httpd listening port
ss -lntp | grep httpd
```

```
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port
...
LISTEN     0      511           *:8080                      *:*                   users:(("httpd",pid=718,fd=3),("httpd",pid=717,fd=3),("httpd",pid=716,fd=3),("httpd",pid=715,fd=3),("httpd",pid=714,fd=3),("httpd",pid=712,fd=3))
...
```

```bash
# View httpd server information
curl -v localhost:8080
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
< Date: Sat, 17 Sep 2022 10:04:37 GMT
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

We can see `Server: Apache/2.4.6 (CentOS) PHP/5.4.16` being displayed.

```bash
# Remove version numbers
cat >> /etc/httpd/conf/httpd.conf
```

```
# Remove server versions
ServerTokens Prod
ServerSignature Off
```

Close the file with control + d i.e. `^D`

```bash
# Restart httpd
systemctl restart httpd

# View server information is off
curl -v localhost:8080
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
< Date: Sat, 17 Sep 2022 10:08:04 GMT
< Server: Apache
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

We can only see `Server: Apache` being displayed.

```bash
# View the /news endpoint
curl -v localhost:8080/news/
```

```
* About to connect() to localhost port 8080 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /news/ HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Sat, 17 Sep 2022 10:09:46 GMT
< Server: Apache
< Content-Length: 880
< Content-Type: text/html;charset=ISO-8859-1
<
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /news</title>
 </head>
 <body>
<h1>Index of /news</h1>
  <table>
   <tr><th valign="top"><img src="/icons/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr>
   <tr><th colspan="5"><hr></th></tr>
<tr><td valign="top"><img src="/icons/back.gif" alt="[PARENTDIR]"></td><td><a href="/">Parent Directory</a>       </td><td>&nbsp;</td><td align="right">  - </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/text.gif" alt="[TXT]"></td><td><a href="news.txt">news.txt</a>               </td><td align="right">2022-09-17 10:01  </td><td align="right">  0 </td><td>&nbsp;</td></tr>
   <tr><th colspan="5"><hr></th></tr>
</table>
</body></html>
* Connection #0 to host localhost left intact
```

We can see things like `news.txt` being displayed.

```bash
# Stop directory contents being listed
vi /etc/httpd/conf/httpd.conf
```

Change `AllowOverride None` to `AllowOverride All` inside of `<Directory /var/www/html/>`

```
<Directory /var/www/html/>
...
  AllowOverride All
...
```

```bash
# Add a .htaccess file to block the folder
cat > /var/www/html/news/.htaccess
```

```
Options -Indexes
```

Close the file with control + d i.e. `^D`


```bash
# Reload httpd and test again
systemctl restart httpd
curl -v localhost:8080/news/
```

```
* About to connect() to localhost port 8080 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /news/ HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
>
< HTTP/1.1 403 Forbidden
< Date: Sat, 17 Sep 2022 10:15:49 GMT
< Server: Apache
< Content-Length: 207
< Content-Type: text/html; charset=iso-8859-1
<
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access /news/
on this server.</p>
</body></html>
* Connection #0 to host localhost left intact
```

We can see `403 Forbidden` being returned which is blocking access.
