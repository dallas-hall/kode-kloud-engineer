# nginx Reverse Proxy

## Task

> Nautilus system admin's team is planning to deploy a front end application for their backup utility on Nautilus Backup Server, so that they can manage the backups of different websites from a graphical user interface. They have shared requirements to set up the same; please accomplish the tasks as per detail given below:
>
> * Install Apache Server on Nautilus Backup Server and configure it to use `8082` port (do not bind it to `127.0.0.1` only, keep it default i.e let Apache listen on server's IP, hostname, localhost, 127.0.0.1 etc).
> * Install Nginx webserver on Nautilus Backup Server and configure it to use `8091`.
> * Configure Nginx as a reverse proxy server for Apache.
> * There is a sample index file `/home/thor/index.html` on Jump Host, copy that file to Apache's document root.
> * Make sure to start Apache and Nginx services.
> * You can test final changes using `curl` command, e.g `curl http://<backup server IP or Hostname>:8091`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* nginx as reverse proxy.
  * https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/
  * https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy-on-ubuntu-22-04

## Steps

```bash
# Copy the HTML file over.
scp /home/thor/index.html clint@stbkp01:/tmp

# Connect to the load balancer server.
ssh clint@stbkp01

# Get root
sudo -i

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Install the web servers and networking tools like `ss`.
dnf install -y httpd nginx iproute

# Copy the HTML file over.
cp /tmp/index.html /var/www/html/

# Take a backup of httpd config.
cp -a /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.old

# Change the port httpd is listening on.
sed -i -r 's/Listen 80/Listen 8082/g' /etc/httpd/conf/httpd.conf

# Enable and start httpd.
systemctl enable --now httpd

# Test httpd from stbkp01 and jump_host.
curl localhost:8082
curl stbkp01:8082
```

```
Welcome to xFusionCorp Industries!
```

It works, move onto nginx.


```bash
# Take a backup of nginx config.
cp -a /etc/nginx/nginx.conf /etc/nginx/nginx.conf.old

# Configure nginx as an RP.
cat > /etc/nginx/nginx.conf
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

    server {
        listen       8091 default_server;
        listen       [::]:8091 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
                proxy_pass http://127.0.0.1:8082;
        }

        error_page 404 /404.html;
            location = /40x.html {
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
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```

Close the file with control + d i.e. `^D`

```bash
# Enable and start nginx.
systemctl enable --now nginx

# Test nginx from stbkkp01 and jump_host.
curl localhost:8091
curl stbkp01:8091
```

```
Welcome to xFusionCorp Industries!
```

It works, we are done.
