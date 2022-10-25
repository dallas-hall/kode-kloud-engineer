# Troubleshooting Failed httpd Process

## Task

> The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems. One of the monitoring systems reported about Apache service unavailability on one of the app servers in Stratos DC.<br><br>Identify the faulty app host and fix the issue. Make sure Apache service is up and running on all app hosts. They might not hosted any code yet on these servers so you need not to worry about if Apache isn't serving any pages or not, just make sure service is up and running. Also, make sure Apache is running on port `6300` on all app servers.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None.

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Check app servers
ssh stapp01 curl -kLv localhost:6300
```

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 6300 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 6300 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:6300
> Accept: */*
> 
{ [data not shown]
100   183    0   183    0     0  26676      0 --:--:-- --:--:-- --:--:-- 30500
* Connection #0 to host localhost left intact
220 stapp01.stratos.xfusioncorp.com ESMTP Sendmail 8.14.7/8.14.7; Tue, 25 Oct 2022 09:01:08 GMT
421 
```

Looks like we have a problem.

```bash
# Connect to the server and investigate
ssh stapp01

# Get root
sudo -i

# Check httpd service
systemctl status httpd
```

```
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd(8)
           man:apachectl(8)
```

Service is dead.

```bash
# Start the server
systemctl enable --now httpd
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
Job for httpd.service failed because the control process exited with error code. See "systemctl status httpd.service" and "journalctl -xe" for details.
```

There is still an error.

```bash
# View the logs
journalctl -xe
```

Searched with `/httpd`.

```
...
-- Unit httpd.service has begun starting up.
Oct 25 09:04:33 stapp01.stratos.xfusioncorp.com httpd[983]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using stapp01.stratos.xf
usioncorp.com. Set the 'ServerName' directive globally to suppress this message
Oct 25 09:04:33 stapp01.stratos.xfusioncorp.com httpd[983]: (98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:6300
Oct 25 09:04:33 stapp01.stratos.xfusioncorp.com httpd[983]: no listening sockets available, shutting down
Oct 25 09:04:33 stapp01.stratos.xfusioncorp.com httpd[983]: AH00015: Unable to open logs
Oct 25 09:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: main process exited, code=exited, status=1/FAILURE
Oct 25 09:04:33 stapp01.stratos.xfusioncorp.com kill[984]: kill: cannot find process ""
Oct 25 09:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: control process exited, code=exited status=1
Oct 25 09:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: Failed to start The Apache HTTP Server.
-- Subject: Unit httpd.service has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit httpd.service has failed.
-- 
-- The result is failed.
Oct 25 09:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: Unit httpd.service entered failed state.
Oct 25 09:04:33 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service failed.
...
```

There is another service listening on that port.

```bash
# View what is running on 6300
ss -lntp | grep 6300
```

```
LISTEN     0      10     127.0.0.1:6300                     *:*                   users:(("sendmail",pid=567,fd=4))
```

```bash
# Stop the service listening on 6300
systemctl disable --now sendmail
```

```
Removed symlink /etc/systemd/system/multi-user.target.wants/sm-client.service.
Removed symlink /etc/systemd/system/multi-user.target.wants/sendmail.service.
```

Could also update the `sendmail` port via `/etc/mail/sendmail.mc` and the default port is 25.

```bash
# Start httpd
systemctl start httpd

# Test
curl -kLv localhost:6300
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
< Date: Tue, 25 Oct 2022 09:08:16 GMT
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

Looks fine. Exit back to the thor jumpbox to check the other servers.

```bash
# Check next server
ssh stapp02 curl -kLv localhost:6300
```

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 6300 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 6300 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:6300
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Tue, 25 Oct 2022 09:01:55 GMT
< Server: Apache/2.4.6 (CentOS) PHP/5.4.16
< Last-Modified: Thu, 30 Nov 2017 23:11:00 GMT
< ETag: "1a4-55f3b5d810900"
< Accept-Ranges: bytes
< Content-Length: 420
< Content-Type: text/html; charset=UTF-8
< 
{ [data not shown]
100   420  100   420    0     0  74<?php
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
812      0 --:--:-- --:--:-- --:--:-- 84000
* Connection #0 to host localhost left intact
```

```bash
# Check next server
ssh stapp03 curl -kLv localhost:6300
```

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 6300 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 6300 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:6300
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Tue, 25 Oct 2022 09:02:25 GMT
< Server: Apache/2.4.6 (CentOS) PHP/5.4.16
< Last-Modified: Thu, 30 Nov 2017 23:11:00 GMT
< ETag: "1a4-55f3b5d810900"
< Accept-Ranges: bytes
< Content-Length: 420
< Content-Type: text/html; charset=UTF-8
< 
{ [data not shown]
100   420  100   420    0     0  74507      0 --:--:-- --:--:-- --:--:-- 84000
* Connection #0 to host localhost left intact
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
```
