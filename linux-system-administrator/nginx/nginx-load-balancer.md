# nginx Reverse Proxy

## Task

> Day by day traffic is increasing on one of the websites managed by the Nautilus production support team. Therefore, the team has observed a degradation in website performance. Following discussions about this issue, the team has decided to deploy this application on a high availability stack i.e on Nautilus infra in Stratos DC. They started the migration last month and it is almost done, as only the LBR server configuration is pending. Configure LBR server as per the information given below:
>
> * Install `nginx` on LBR (load balancer) server.
> * Configure load-balancing with the `http` context making use of all App Servers.
> * Also make sure Apache service is up and running on all app servers.
> * Once done, you can access the website using StaticApp button on the top bar.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* nginx as load balancer
  * https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/
  * https://nginx.org/en/docs/http/load_balancing.html

## Steps


Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Check httpd is running on all stapp servers from the jumpbox
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03

# Check current Linux version, it was CentOS Stream 8.
cat /etc/*rel*

# Switch to root
sudo -i

# Check httpd, it was running on port 6300.
systemctl status httpd

# Connect to the app server from the jumpbox.
ssh loki@stlb01

# Check current Linux version, it was CentOS Stream 8.
cat /etc/*rel*

# Switch to root
sudo -i

# Install nginx
dnf install -y nginx

# Enable and start nginx
systemctl enable --now nginx

# Edit /etc/nginx/nginx.conf and update it to the content below.
cp -a /etc/nginx/nginx.conf /etc/nginx/nginx.conf.old
vi /etc/nginx/nginx.conf
```

```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

   upstream stapp {
       server stapp01.stratos.xfusioncorp.com:6300;
       server stapp02.stratos.xfusioncorp.com:6300;
       server stapp03.stratos.xfusioncorp.com:6300;
   }

   server {
       listen 80;

       location / {
           proxy_pass http://stapp;
       }
   }
}
```

```bash
# Reload nginx config and restart
systemctl reload nginx
systemctl restart nginx

# Check the load balancer is working from the jumpbox.
curl -v stlb01
```

```
* Rebuilt URL to: stlb01/
*   Trying 172.16.238.14...
* TCP_NODELAY set
* Connected to stlb01 (172.16.238.14) port 80 (#0)
> GET / HTTP/1.1
> Host: stlb01
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.14.1
< Date: Mon, 05 Feb 2024 06:06:29 GMT
< Content-Type: text/html; charset=UTF-8
< Content-Length: 34
< Connection: keep-alive
< Last-Modified: Mon, 05 Feb 2024 05:42:55 GMT
< ETag: "22-6109beef0d5c0"
< Accept-Ranges: bytes
<
* Connection #0 to host stlb01 left intact
Welcome to xFusionCorp Industries!

```

Clicking the button shows the same rsult. It works, we are done.
