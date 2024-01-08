# httpd Password Protected Folder With PAM

## Task

> We have a requirement where we want to password protect a directory in the Apache web server document root. We want to password protect `http://<website-url>:<apache_port>/protected` URL as per the following requirements (you can use any website-url for it like localhost since there are no such specific requirements as of now). Setup the same on App server 3 as per below mentioned requirements:
>
> * We want to use basic authentication.
> * We do not want to use `htpasswd` file based authentication. Instead, we want to use PAM authentication, i.e Basic Auth + PAM so that we can authenticate with a Linux user.
> * We already have a user `kareem` with password `LQfKeWWxWD` which you need to provide access to.
> * You can test the same using a `curl` command from jump host `curl http://<website-url>:<apache_port>/protected`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* httpd basic auth packages
  * https://cwiki.apache.org/confluence/display/HTTPD/PasswordBasicAuth
  * https://www.server-world.info/en/note?os=CentOS_7&p=httpd&f=10

## Steps

```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS 7.6
cat /etc/*rel*

# Switch to root
sudo -i

# Check if Apache httpd is running, it wasn't.
systemctl status httpd

# Start and enable httpd.
systemctl enable --now httpd


# Find the httpd port, it was 8080.
ss -lntp | grep http
```

```
LISTEN     0      511          *:8080                     *:*                   users:(("httpd",pid=773,fd=3),("httpd",pid=772,fd=3),("httpd",pid=771,fd=3),("httpd",pid=770,fd=3),("httpd",pid=769,fd=3),("httpd",pid=768,fd=3))
```

```bash
# Test local connectivity, it worked.
curl -Lv http://localhost:8080/

# Test local connectivity to protected, it worked without basic auth.
curl -Lv http://localhost:8080/protected

# Install the necessary packages
yum --enablerepo=epel -y install mod_authnz_external pwauth

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

# Test local connectivity, it worked.
curl -Lv http://localhost:8080/

# Test local connectivity to protected, it failed as expected without basic auth.
curl -Lv http://localhost:8080/protected
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

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
< Date: Mon, 08 Jan 2024 05:54:33 GMT
< Server: Apache/2.4.6 (CentOS) PHP/7.2.26
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

</details>

```bash
# Test local connectivity to protected, it worked as expected with basic auth.
curl -Lv --basic --user kareem:LQfKeWWxWD localhost:8080/protected
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* About to connect() to localhost port 8080 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
* Server auth using Basic with user 'kareem'
> GET /protected HTTP/1.1
> Authorization: Basic a2FyZWVtOkxRZktlV1d4V0Q=
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< Date: Mon, 08 Jan 2024 05:55:02 GMT
< Server: Apache/2.4.6 (CentOS) PHP/7.2.26
< Location: http://localhost:8080/protected/
< Content-Length: 240
< Content-Type: text/html; charset=iso-8859-1
< 
* Ignoring the response-body
* Connection #0 to host localhost left intact
* Issue another request to this URL: 'http://localhost:8080/protected/'
* Found bundle for host localhost: 0x14ddeb0
* Re-using existing connection! (#0) with host localhost
* Connected to localhost (127.0.0.1) port 8080 (#0)
* Server auth using Basic with user 'kareem'
> GET /protected/ HTTP/1.1
> Authorization: Basic a2FyZWVtOkxRZktlV1d4V0Q=
> User-Agent: curl/7.29.0
> Host: localhost:8080
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Mon, 08 Jan 2024 05:55:02 GMT
< Server: Apache/2.4.6 (CentOS) PHP/7.2.26
< Last-Modified: Mon, 08 Jan 2024 05:49:32 GMT
< ETag: "26-60e68c31937f8"
< Accept-Ranges: bytes
< Content-Length: 38
< Content-Type: text/html; charset=UTF-8
< 
This is KodeKloud Protected Directory
* Connection #0 to host localhost left intact
```

</details>

```bash
# Test external connectivity from the jump host, it worked.
exit
exit
curl -Lv http://stapp03:8080/

# Test external connectivity from the jump host, it failed as expected without basic auth.
curl -Lv http://stapp03:8080/protected

# Test external connectivity from the jump host, it worked as expected with basic auth.
curl -Lv --basic --user kareem:LQfKeWWxWD http://stapp03:8080/protected
```

It is working, we are done.
