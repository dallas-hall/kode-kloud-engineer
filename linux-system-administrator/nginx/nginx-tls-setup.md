# nginx TLS

## Task

> The system admins team of xFusionCorp Industries needs to deploy a new application on App Server 3 in Stratos Datacenter. They have some pre-requites to get ready that server for application deployment. Prepare the server as per requirements shared below:
>
> * Install and configure nginx on App Server 2.
> * On App Server 3 there is a self signed SSL certificate and key present at location `/tmp/nautilus.crt` and `/tmp/nautilus.key`. Move them to some appropriate location and deploy the same in Nginx.
> * Create an `index.html` file with content `Welcome!` under Nginx document root.
>
> For final testing try to access the App Server 3 link (either hostname or IP) from jump host using `curl` command. For example `curl -Ik https://<app-server-ip>/`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Securing nginx
  * https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-8
  * https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-8
  * https://nginx.org/en/docs/http/configuring_https_servers.html

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Connect to the app server.
ssh banner@stapp03

# Check current Linux version, it was CentOS Stream 8.
cat /etc/*rel*

# Switch to root
sudo -i

# Install nginx
dnf install -y nginx

# Enable and start nginx
systemctl enable --now nginx

# Install networking tools, e.g. ss
dnf install -y iproute

# Check the listening ports.
ss -lntp | grep nginx
```

```
State    Recv-Q   Send-Q     Local Address:Port        Peer Address:Port   Process
...
LISTEN   0        511              0.0.0.0:80               0.0.0.0:*       users:(("nginx",pid=1836,fd=6), ...)
LISTEN   0        511                 [::]:80                  [::]:*       users:(("nginx",pid=1836,fd=7), ...)
...
```

We can see `nginx` listening for http on port 80.

```bash
# Make the certificate and private key folders
mkdir -p -m 755 /etc/pki/nginx/private

# Copy certificate and private key
cp /tmp/nautilus.crt /etc/pki/nginx/
cp /tmp/nautilus.key /etc/pki/nginx/private/

# Edit /etc/nginx/nginx.conf and uncomment out the server block for TLS
cp -a /etc/nginx/nginx.conf /etc/nginx/nginx.conf.old
vi /etc/nginx/nginx.conf
```

```
...
# Settings for a TLS enabled server.
#
   server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/etc/pki/nginx/nautilus.crt";
        ssl_certificate_key "/etc/pki/nginx/private/nautilus.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
...
```

```bash
# Reload nginx config and restart
systemctl reload nginx
systemctl restart nginx

# Check the listening ports for 80 and 443
ss -lntp
```

```
State    Recv-Q   Send-Q     Local Address:Port        Peer Address:Port   Process
...
LISTEN   0        511              0.0.0.0:443              0.0.0.0:*       users:(("nginx",pid=1836,fd=8), ...)
LISTEN   0        511              0.0.0.0:80               0.0.0.0:*       users:(("nginx",pid=1836,fd=6), ...)
LISTEN   0        511                 [::]:443                 [::]:*       users:(("nginx",pid=1836,fd=9), ...)
LISTEN   0        511                 [::]:80                  [::]:*       users:(("nginx",pid=1836,fd=7), ...)
...
```

We can see `nginx` listening for http on port 80 and https on port 443.

```bash
# Create the index.html file.
cat > /usr/share/nginx/html/index.html
```

```
Welcome!
```

Close the file with control + d i.e. `^D`


```bash
# Test http locally
curl -kLv http://localhost
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* Rebuilt URL to: http://localhost/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: localhost
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.14.1
< Date: Wed, 10 Jan 2024 04:46:11 GMT
< Content-Type: text/html
< Content-Length: 9
< Last-Modified: Wed, 10 Jan 2024 04:46:03 GMT
< Connection: keep-alive
< ETag: "659e210b-9"
< Accept-Ranges: bytes
<
Welcome!
* Connection #0 to host localhost left intact
```

</details>

```bash
# Test https locally
curl -kLv https://localhost
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* Rebuilt URL to: https://localhost/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, [no content] (0):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=SP; ST=SINGAPORE; L=SINGAPORE; O=KODEKLOUD; CN=stlb01.stratos.xfusioncorp.com; emailAddress=mmumshad@kodekloud.com
*  start date: Jan 20 14:29:58 2020 GMT
*  expire date: Jan 17 14:29:58 2030 GMT
*  issuer: C=SP; ST=SINGAPORE; L=SINGAPORE; O=KODEKLOUD; CN=stlb01.stratos.xfusioncorp.com; emailAddress=mmumshad@kodekloud.com
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* TLSv1.3 (OUT), TLS app data, [no content] (0):
* TLSv1.3 (OUT), TLS app data, [no content] (0):
* TLSv1.3 (OUT), TLS app data, [no content] (0):
* Using Stream ID: 1 (easy handle 0x558cada646f0)
* TLSv1.3 (OUT), TLS app data, [no content] (0):
> GET / HTTP/2
> Host: localhost
> User-Agent: curl/7.61.1
> Accept: */*
>
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS app data, [no content] (0):
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
* TLSv1.3 (OUT), TLS app data, [no content] (0):
* TLSv1.3 (IN), TLS app data, [no content] (0):
* TLSv1.3 (IN), TLS app data, [no content] (0):
< HTTP/2 200
< server: nginx/1.14.1
< date: Wed, 10 Jan 2024 04:46:28 GMT
< content-type: text/html
< content-length: 9
< last-modified: Wed, 10 Jan 2024 04:46:03 GMT
< etag: "659e210b-9"
< accept-ranges: bytes
<
Welcome!
* Connection #0 to host localhost left intact
```

</details>

```bash
# Test http from thor jumpbox
curl -kLv http://stapp03
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* Rebuilt URL to: http://stapp03/
*   Trying 172.16.238.12...
* TCP_NODELAY set
* Connected to stapp03 (172.16.238.12) port 80 (#0)
> GET / HTTP/1.1
> Host: stapp03
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.14.1
< Date: Wed, 10 Jan 2024 04:46:54 GMT
< Content-Type: text/html
< Content-Length: 9
< Last-Modified: Wed, 10 Jan 2024 04:46:03 GMT
< Connection: keep-alive
< ETag: "659e210b-9"
< Accept-Ranges: bytes
<
Welcome!
* Connection #0 to host stapp03 left intact
```

</details>

```bash
# Test https from thor jumpbox
curl -kLv https://stapp03
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
* Rebuilt URL to: https://stapp03/
*   Trying 172.16.238.12...
* TCP_NODELAY set
* Connected to stapp03 (172.16.238.12) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, [no content] (0):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=SP; ST=SINGAPORE; L=SINGAPORE; O=KODEKLOUD; CN=stlb01.stratos.xfusioncorp.com; emailAddress=mmumshad@kodekloud.com
*  start date: Jan 20 14:29:58 2020 GMT
*  expire date: Jan 17 14:29:58 2030 GMT
*  issuer: C=SP; ST=SINGAPORE; L=SINGAPORE; O=KODEKLOUD; CN=stlb01.stratos.xfusioncorp.com; emailAddress=mmumshad@kodekloud.com
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* TLSv1.3 (OUT), TLS app data, [no content] (0):
* TLSv1.3 (OUT), TLS app data, [no content] (0):
* TLSv1.3 (OUT), TLS app data, [no content] (0):
* Using Stream ID: 1 (easy handle 0x55f4a94156b0)
* TLSv1.3 (OUT), TLS app data, [no content] (0):
> GET / HTTP/2
> Host: stapp03
> User-Agent: curl/7.61.1
> Accept: */*
>
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS app data, [no content] (0):
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
* TLSv1.3 (OUT), TLS app data, [no content] (0):
* TLSv1.3 (IN), TLS app data, [no content] (0):
< HTTP/2 200
< server: nginx/1.14.1
< date: Wed, 10 Jan 2024 04:47:12 GMT
< content-type: text/html
< content-length: 9
< last-modified: Wed, 10 Jan 2024 04:46:03 GMT
< etag: "659e210b-9"
< accept-ranges: bytes
<
Welcome!
* Connection #0 to host stapp03 left intact
```

</details>

We are done.
