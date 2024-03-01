# Install LAMP Stack

## Task

> xFusionCorp Industries is planning to host a WordPress website on their infra in Stratos Datacenter. They have already done infrastructure configuration—for example, on the storage server they already have a shared directory `/vaw/www/html` that is mounted on each app host under `/var/www/html` directory. Please perform the following steps to accomplish the task:
>
> * Install `httpd`, `php`and its dependencies on all app hosts.
> * Apache should serve on port `5001` within the apps.
> * Install/Configure MariaDB server on DB Server.
> * Create a database named `kodekloud_db7` and create a database user named `kodekloud_sam` identified as password `TmPcZjtRQx`. Further make sure this newly created user is able to perform all operation on the database you created.
> * Finally you should be able to access the website on LBR link, by clicking on the App button on the top bar. You should see a message like App is able to connect to the database using user `kodekloud_sam`

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mariadb-php-lamp-stack-on-centos-8

## Steps

```bash
# Install required packages and configure httpd on all app servers.
ssh -t tony@stapp01 'sudo dnf install -y httpd php php-mysqlnd && sudo cp -a /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.old && sudo sed -i -r "s/Listen 80/Listen 5001/g" /etc/httpd/conf/httpd.conf && sudo systemctl enable --now httpd'
ssh -t steve@stapp02 'sudo dnf install -y httpd php php-mysqlnd && sudo cp -a /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.old && sudo sed -i -r "s/Listen 80/Listen 5001/g" /etc/httpd/conf/httpd.conf && sudo systemctl enable --now httpd'
ssh -t banner@stapp03 'sudo dnf install -y httpd php php-mysqlnd && sudo cp -a /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.old && sudo sed -i -r "s/Listen 80/Listen 5001/g" /etc/httpd/conf/httpd.conf && sudo systemctl enable --now httpd'

# Connect to the database server
ssh peter@stdb01

# Switch to root
sudo -i

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Install useful tools, e.g. ss, nslookup, clear, and required packages
dnf install -y iproute bind-utils ncurses mariadb-server

# Enable and start mariadb
systemctl enable --now mariadb

# Set up the database
export DB_NAME=kodekloud_db7
export DB_USER=kodekloud_sam
export DB_PASSWORD=TmPcZjtRQx

mysql -e "create database if not exists $DB_NAME;"
mysql -e "CREATE USER IF NOT EXISTS '$DB_USER' IDENTIFIED BY '$DB_PASSWORD';"
mysql -e "GRANT ALL ON $DB_NAME.* TO '$DB_USER';"
mysql --user="$DB_USER" --password="$DB_PASSWORD" --database="$DB_NAME" -e 'SELECT VERSION(); SELECT DATABASE(); SELECT USER();'
```

```
+-----------------+
| VERSION()       |
+-----------------+
| 10.3.28-MariaDB |
+-----------------+
+---------------+
| DATABASE()    |
+---------------+
| kodekloud_db7 |
+---------------+
+-------------------------+
| USER()                  |
+-------------------------+
| kodekloud_sam@localhost |
+-------------------------+
```

```bash
# Test connectivity from the jump box.
curl stapp01:5001
curl stapp02:5001
curl stapp03:5001
```

```
App is able to connect to the database using user kodekloud_sam
```

It is working, clicking the App button works too. We are done.
