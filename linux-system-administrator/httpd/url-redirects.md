# URL Redirects

## Task

> The Nautilus devops team got some requirements related to some Apache config changes. They need to setup some redirects for some URLs. There might be some more changes need to be done. Below you can find more details regarding that:
> 
> * `httpd` is already installed on app server 1. Configure Apache to listen on port `8084`.
> * Configure Apache to add some redirects as mentioned below:
>   * Redirect `http://stapp01.stratos.xfusioncorp.com:<Port>/` to `http://www.stapp01.stratos.xfusioncorp.com:<Port>/` i.e non www to www. This must be a permanent redirect i.e `301`<br>
>   * Redirect `http://www.stapp01.stratos.xfusioncorp.com:<Port>/blog/` to `http://www.stapp01.stratos.xfusioncorp.com:<Port>/news/`. This must be a temporary redirect i.e `302`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* httpd redirecting
  * https://www.digitalocean.com/community/tutorials/how-to-create-temporary-and-permanent-redirects-with-apache-and-nginx
  * https://www.linode.com/docs/guides/redirect-urls-with-the-apache-web-server/
  * https://httpd.apache.org/docs/2.4/rewrite/remapping.html
  * https://httpd.apache.org/docs/2.4/mod/mod_alias.html#redirect

## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
cat /etc/*rel*

# Switch to root
sudo -i

# Check which port https is listening on
grep -i listen /etc/httpd/conf/httpd.conf
```

```
...
Listen 8080
```

```bash
# Take a backup
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.old

# Change the port httpd is listening on
sed -i -r 's/Listen 8080/Listen 8084/g' /etc/httpd/conf/httpd.conf

# Start and enable the httpd service
systemctl enable --now httpd

# Check httpd
curl -Lv http://stapp01.stratos.xfusioncorp.com:8084/
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* About to connect() to stapp01.stratos.xfusioncorp.com port 8084 (#0)
*   Trying 172.16.238.10...
* Connected to stapp01.stratos.xfusioncorp.com (172.16.238.10) port 8084 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: stapp01.stratos.xfusioncorp.com:8084
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Mon, 04 Dec 2023 05:54:53 GMT
< Server: Apache/2.4.6 (CentOS) PHP/7.2.26
< Last-Modified: Mon, 04 Dec 2023 05:49:57 GMT
< ETag: "1f-60ba8b017d548"
< Accept-Ranges: bytes
< Content-Length: 31
< Content-Type: text/html; charset=UTF-8
< 
Welcome to the Nautilus Group!
* Connection #0 to host stapp01.stratos.xfusioncorp.com left intact
```

</details>

```bash
# Add the http response headers at the end of the file
cat >> /etc/httpd/conf/httpd.conf
```

```
# Adding custom redirects
Redirect 302 "/blog" "/news"

<VirtualHost *:8084>
  ServerName stapp01.stratos.xfusioncorp.com
  Redirect 301 "/" "http://www.stapp01.stratos.xfusioncorp.com:8084"
</VirtualHost>

<VirtualHost *:8084>
  ServerName www.stapp01.stratos.xfusioncorp.com
</VirtualHost>
```

Close the file with control + d i.e. `^D`

```bash
# Reload httpd to pick up the new configuration
systemctl reload httpd

# Test 200 changes
curl -Lv www.stapp01.stratos.xfusioncorp.com:8084/
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* About to connect() to www.stapp01.stratos.xfusioncorp.com port 8084 (#0)
*   Trying 172.16.238.10...
* Connected to www.stapp01.stratos.xfusioncorp.com (172.16.238.10) port 8084 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: www.stapp01.stratos.xfusioncorp.com:8084
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Mon, 04 Dec 2023 05:56:35 GMT
< Server: Apache/2.4.6 (CentOS) PHP/7.2.26
< Last-Modified: Mon, 04 Dec 2023 05:49:57 GMT
< ETag: "1f-60ba8b017d548"
< Accept-Ranges: bytes
< Content-Length: 31
< Content-Type: text/html; charset=UTF-8
< 
Welcome to the Nautilus Group!
* Connection #0 to host www.stapp01.stratos.xfusioncorp.com left intact
```

</details>

We can see a direct 200 connect.

```bash
# Test 301 changes
curl -Lv stapp01.stratos.xfusioncorp.com:8084/
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* About to connect() to stapp01.stratos.xfusioncorp.com port 8084 (#0)
*   Trying 172.16.238.10...
* Connected to stapp01.stratos.xfusioncorp.com (172.16.238.10) port 8084 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: stapp01.stratos.xfusioncorp.com:8084
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< Date: Mon, 04 Dec 2023 05:56:58 GMT
< Server: Apache/2.4.6 (CentOS) PHP/7.2.26
< Location: http://www.stapp01.stratos.xfusioncorp.com:8084
< Content-Length: 255
< Content-Type: text/html; charset=iso-8859-1
< 
* Ignoring the response-body
* Connection #0 to host stapp01.stratos.xfusioncorp.com left intact
* Issue another request to this URL: 'http://www.stapp01.stratos.xfusioncorp.com:8084'
* About to connect() to www.stapp01.stratos.xfusioncorp.com port 8084 (#1)
*   Trying 172.16.238.10...
* Connected to www.stapp01.stratos.xfusioncorp.com (172.16.238.10) port 8084 (#1)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: www.stapp01.stratos.xfusioncorp.com:8084
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Mon, 04 Dec 2023 05:56:58 GMT
< Server: Apache/2.4.6 (CentOS) PHP/7.2.26
< Last-Modified: Mon, 04 Dec 2023 05:49:57 GMT
< ETag: "1f-60ba8b017d548"
< Accept-Ranges: bytes
< Content-Length: 31
< Content-Type: text/html; charset=UTF-8
< 
Welcome to the Nautilus Group!
* Connection #1 to host www.stapp01.stratos.xfusioncorp.com left intact
```

</details>

We can see a 301 permanent redirect and then a 200.

```bash
# Test 302 changes
curl -Lv www.stapp01.stratos.xfusioncorp.com:8084/blog/
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* About to connect() to www.stapp01.stratos.xfusioncorp.com port 8084 (#0)
*   Trying 172.16.238.10...
* Connected to www.stapp01.stratos.xfusioncorp.com (172.16.238.10) port 8084 (#0)
> GET /blog/ HTTP/1.1
> User-Agent: curl/7.29.0
> Host: www.stapp01.stratos.xfusioncorp.com:8084
> Accept: */*
> 
< HTTP/1.1 302 Found
< Date: Mon, 04 Dec 2023 05:59:34 GMT
< Server: Apache/2.4.6 (CentOS) PHP/7.2.26
< Location: http://www.stapp01.stratos.xfusioncorp.com:8084/news/
< Content-Length: 237
< Content-Type: text/html; charset=iso-8859-1
< 
* Ignoring the response-body
* Connection #0 to host www.stapp01.stratos.xfusioncorp.com left intact
* Issue another request to this URL: 'http://www.stapp01.stratos.xfusioncorp.com:8084/news/'
* Found bundle for host www.stapp01.stratos.xfusioncorp.com: 0x15ffeb0
* Re-using existing connection! (#0) with host www.stapp01.stratos.xfusioncorp.com
* Connected to www.stapp01.stratos.xfusioncorp.com (172.16.238.10) port 8084 (#0)
> GET /news/ HTTP/1.1
> User-Agent: curl/7.29.0
> Host: www.stapp01.stratos.xfusioncorp.com:8084
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Mon, 04 Dec 2023 05:59:34 GMT
< Server: Apache/2.4.6 (CentOS) PHP/7.2.26
< Last-Modified: Mon, 04 Dec 2023 05:59:30 GMT
< ETag: "24-60ba8d242c495"
< Accept-Ranges: bytes
< Content-Length: 36
< Content-Type: text/html; charset=UTF-8
< 
Welcome to the Nautilus Group NEWS!
* Connection #0 to host www.stapp01.stratos.xfusioncorp.com left intact
```

</details>

Initially this was 404 not found. I had to `mkdir -p /var/www/html/news/` and `cat > /var/www/html/news/index.html` to create the redirect folder and content. After that we can see a 302 temporary redirect and then a 200.

We are done.
