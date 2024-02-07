# Troubleshooting LEMP Stack

## Task

> We have LEMP stack configured on apps and database server in Stratos DC. Its using Nginx + php-fpm, for now we have deployed a sample php page on these apps. Due to some misconfiguration the php page is not loading on the web server. Seems like at least two app servers are having issues. Find below more details and make sure website works on LBR URL and locally on each app as well.
> * Nginx is supposed to run on port `80` on all app servers.
> * Nginx document root is `/var/www/html/`
> * Test the webpage on LBR URL (use LBR button on the top bar) and locally on each app server to make sure it works. It must not display any error message or `nginx` default page.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None.

## Steps

```bash
# Connect to the load balancer server
ssh loki@stlb01

# Switch to root
sudo -i

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Install useful tools, e.g. ss, nslookup, clear
dnf install -y iproute bind-utils ncurses

# View listening services. HAProxy was listening on 80.
ss -lntp

# Test local connection, it showed the default nginx page which is wrong according to above.
curl -v localhost

# View the HAProxy config.
cat /etc/haproxy/haproxy.cfg
```

```
global
  maxconn 4096
  pidfile /var/run/haproxy.pid
defaults
  mode http
  retries 3
  option redispatch
  maxconn 2000
  timeout connect 5000
  timeout check 5000
  timeout client 30000
  timeout server 30000

frontend myAppFrontEnd
  bind *:${BIND_PORT}
  use_backend webservers
backend webservers
    balance roundrobin
    server web1 ${WEB_APP1}:${WEB_APP1_PORT} check
    server web2 ${WEB_APP2}:${WEB_APP2_PORT} check
    server web3 ${WEB_APP3}:${WEB_APP3_PORT} check
```

At first glance this looks okay.

```bash
# Connect to the app servers from the jumpbox.
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03

# Check current Linux version, it was CentOS Stream 8.
cat /etc/*rel*

# Switch to root
sudo -i

# Install useful tools, e.g. ss, nslookup, clear
dnf install -y iproute bind-utils ncurses

# View listening services. nginx was listening on 8084.
ss -lntp

# Check the /var/www/html contents.
ls -Ahl /var/www/html
```

```
-rw-r--r-- 1 root root 267 Feb  7 04:07 index.php
```

```bash
# Fix nginx config.
cp -a /etc/nginx/nginx.conf /etc/nginx/nginx.conf.old
vi /etc/nginx/nginx.conf
```

And make the contents:

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

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /var/www/html/;
        index index.html index.htm index.php;


        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

       location ~ .php$ {
           try_files $uri =404;
           fastcgi_pass unix:/var/opt/remi/php74/run/php-fpm/www.sock;
           fastcgi_index index.php;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           include fastcgi_params;
        }
        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2 default_server;
#        listen       [::]:443 ssl http2 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers HIGH:!aNULL:!MD5;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#        location = /404.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#        location = /50x.html {
#        }
#    }

}
```

I ended up changing the port number to 80, adding in `index.php` to `index`, removing `.sock` from `fastcgi_pass unix:/var/opt/remi/php74/run/php-fpm/www.sock`, and setting the `root` to `/var/www/html`.

Close the file `:x`


```bash
# Reload nginx config and restart
systemctl reload nginx
systemctl restart nginx

# Test the changes, it was okay.
curl -v localhost
```

```
App is able to connect to the database using user kodekloud_aim
```

Test using the load balancer from the jumpbox via `curl stlb01.stratos.xfusioncorp.com` and the LBR button. They both had the same result as above.

We are done.
