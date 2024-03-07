# nginx & php-fpm With Unix Socket

## Task

> The Nautilus application development team is planning to launch a new PHP-based application, which they want to deploy on Nautilus infra in Stratos DC. The development team had a meeting with the production support team and they have shared some requirements regarding the infrastructure. Below are the requirements they shared:
>
> * Install `nginx` on app server 2 , configure it to use port `8097` and its document root should be `/var/www/html`.
> * Install `php-fpm` version 7.2 on app server 2, it must use the unix socket `/var/run/php-fpm/default.sock` (create the parent directories if don't exist).
> * Configure `php-fpm` and nginx to work together.
> * Once configured correctly, you can test the website using `curl http://stapp02:8097/index.php` command from jump host.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* nginx
  * https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-8
* php-fpm
  * https://linuxize.com/post/how-to-install-php-on-centos-8/
  * https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-centos-8
  * https://www.tecmint.com/connect-nginx-to-php-fpm/

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Connect to the app server.
ssh steve@stapp02

# Check current Linux version, it was CentOS Stream 8.
cat /etc/*rel*

# Switch to root
sudo -i

# Install useful tools, e.g. ss, nslookup, clear, scp, and required packages
dnf install -y iproute bind-utils ncurses bash-completion openssh-clients less diffutils nginx

# Change nginx listening port and document root
cp -a /etc/nginx/nginx.conf /etc/nginx/nginx.conf.old
sed -i -r 's/80/8097/g' /etc/nginx/nginx.conf
sed -i -r 's|/usr/share/nginx/html|/var/www/html|g' /etc/nginx/nginx.conf

# Install php-fpm 7.2 which is the defatul php in CentOS 8.
dnf install -y php

# Change php-fpm user:group permissions.
cp -a /etc/php-fpm.d/www.conf /etc/php-fpm.d/www.conf.old
sed -i -r 's/user = apache/user = nginx/g' /etc/php-fpm.d/www.conf
sed -i -r 's/group = apache/group = nginx/g' /etc/php-fpm.d/www.conf

# Change php-fpm listening socket.
mkdir -p /var/run/php-fpm
touch /var/run/php-fpm/default.sock
chown nginx: /var/run/php-fpm/default.sock
sed -i -r 's|listen = /run/php-fpm/www.sock|listen = /var/run/php-fpm/default.sock|g' /etc/php-fpm.d/www.conf
sed -i -r 's/;listen.owner = nobody/listen.owner = nginx/g' /etc/php-fpm.d/www.conf
sed -i -r 's/;listen.group = nobody/listen.group = ngnix/g' /etc/php-fpm.d/www.conf
sed -i -r 's/;listen.mode = 0660/listen.mode = 0660/g' /etc/php-fpm.d/www.conf

# Change nginx to listen on the Unix socket.
cp -a /etc/nginx/conf.d/php-fpm.conf /etc/nginx/conf.d/php-fpm.conf.old
sed -i -r 's|server unix:/run/php-fpm/www.sock;|server unix:/var/run/php-fpm/default.sock;|g' /etc/nginx/conf.d/php-fpm.conf

# Enable and start nginx and php-fpm
systemctl enable --now nginx php-fpm

# Test locally and remotely from thor.
curl localhost:8097/index.php
exit
curl http://stapp02:8097/index.php
```

```
Welcome to xFusionCorp Industries!
```

It is working, we are done.
