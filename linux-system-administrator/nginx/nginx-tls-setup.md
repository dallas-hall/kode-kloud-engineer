# nginx TLS

## Task

> The system admins team of xFusionCorp Industries needs to deploy a new application on App Server 2 in Stratos Datacenter. They have some pre-requites to get ready that server for application deployment. Prepare the server as per requirements shared below:<br><br>Install and configure nginx on App Server 2.<br>On App Server 2 there is a self signed SSL certificate and key present at location `/tmp/nautilus.crt` and `/tmp/nautilus.key`. Move them to some appropriate location and deploy the same in Nginx.<br>Create an `index.html` file with content `Welcome!` under Nginx document root.<br><br>For final testing try to access the App Server 2 link (either hostname or IP) from jump host using `curl` command. For example `curl -Ik https://<app-server-ip>/`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Install EPEL - https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-on-centos-7
* Securing nginx - https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7 & https://nginx.org/en/docs/http/configuring_https_servers.html

## Steps

```bash
# Check the architecture map - https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0

# Connect to thor jumpbox

# Connect to application servers, logins at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus
ssh steve@stapp02

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Install the EPEL to get the nginx package
yum install -y epel-release
```

```
...
Installed:
  epel-release.noarch 0:7-11

Complete!
```

```bash
# Install nginx
yum install -y nginx
```

```
...
Installed:
  nginx.x86_64 1:1.20.1-9.el7
...
```

```bash
# Enable and start nginx
systemctl enable --now nginx

# Check the listening ports
ss -lntp | grep nginx
```

```
State       Recv-Q Send-Q                                        Local Address:Port                                                       Peer Address:Port
...
LISTEN      0      511                                                       *:80                                                                    *:*                   users:("nginx"...)
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
State       Recv-Q Send-Q                                        Local Address:Port                                                       Peer Address:Port
LISTEN      0      511                                                       *:443                                                                   *:*                   users:(("nginx",pid=1336,fd=8),("nginx"...)
LISTEN      0      511                                                       *:80                                                                    *:*                   users:(("nginx",pid=1336,fd=6),("nginx"...)
...
```

We can see `nginx` listening for http on port 80 and https on port 443.

```bash
# Create the index.html path and file
mkdir -p /usr/share/doc/HTML
cat > /usr/share/doc/HTML/index.html
```

```
Welcome!
```

Close the file with control + d i.e. `^D`


```bash
# Test http locally
curl -kLv http://localhost
```

```
* About to connect() to localhost port 80 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.20.1
< Date: Wed, 14 Sep 2022 09:36:31 GMT
< Content-Type: text/html
< Content-Length: 9
< Last-Modified: Wed, 14 Sep 2022 09:35:08 GMT
< Connection: keep-alive
< ETag: "6321a04c-9"
< Accept-Ranges: bytes
<
Welcome!
* Connection #0 to host localhost left intact
```

```bash
# Test https locally
curl -kLv https://localhost
```

```
* About to connect() to localhost port 443 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 443 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
* skipping SSL peer certificate verification
* SSL connection using TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* Server certificate:
*       subject: E=mmumshad@kodekloud.com,CN=stlb01.stratos.xfusioncorp.com,O=KODEKLOUD,L=SINGAPORE,ST=SINGAPORE,C=SP
*       start date: Jan 20 14:29:58 2020 GMT
*       expire date: Jan 17 14:29:58 2030 GMT
*       common name: stlb01.stratos.xfusioncorp.com
*       issuer: E=mmumshad@kodekloud.com,CN=stlb01.stratos.xfusioncorp.com,O=KODEKLOUD,L=SINGAPORE,ST=SINGAPORE,C=SP
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.20.1
< Date: Wed, 14 Sep 2022 09:36:51 GMT
< Content-Type: text/html
< Content-Length: 9
< Last-Modified: Wed, 14 Sep 2022 09:35:08 GMT
< Connection: keep-alive
< ETag: "6321a04c-9"
< Accept-Ranges: bytes
<
Welcome!
* Connection #0 to host localhost left intact
```

```bash
# Test http from thor jumpbox
curl -kLv http://stapp02
```

```
* About to connect() to stapp02 port 80 (#0)
*   Trying 172.16.238.11...
* Connected to stapp02 (172.16.238.11) port 80 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: stapp02
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.20.1
< Date: Wed, 14 Sep 2022 09:37:59 GMT
< Content-Type: text/html
< Content-Length: 9
< Last-Modified: Wed, 14 Sep 2022 09:35:08 GMT
< Connection: keep-alive
< ETag: "6321a04c-9"
< Accept-Ranges: bytes
<
Welcome!
* Connection #0 to host stapp02 left intact
```

```bash
# Test https from thor jumpbox
curl -kLv https://stapp02
```

```
* About to connect() to stapp02 port 443 (#0)
*   Trying 172.16.238.11...
* Connected to stapp02 (172.16.238.11) port 443 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
* skipping SSL peer certificate verification
* SSL connection using TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* Server certificate:
*       subject: E=mmumshad@kodekloud.com,CN=stlb01.stratos.xfusioncorp.com,O=KODEKLOUD,L=SINGAPORE,ST=SINGAPORE,C=SP
*       start date: Jan 20 14:29:58 2020 GMT
*       expire date: Jan 17 14:29:58 2030 GMT
*       common name: stlb01.stratos.xfusioncorp.com
*       issuer: E=mmumshad@kodekloud.com,CN=stlb01.stratos.xfusioncorp.com,O=KODEKLOUD,L=SINGAPORE,ST=SINGAPORE,C=SP
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: stapp02
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.20.1
< Date: Wed, 14 Sep 2022 09:38:14 GMT
< Content-Type: text/html
< Content-Length: 9
< Last-Modified: Wed, 14 Sep 2022 09:35:08 GMT
< Connection: keep-alive
< ETag: "6321a04c-9"
< Accept-Ranges: bytes
<
Welcome!
* Connection #0 to host stapp02 left intact
```