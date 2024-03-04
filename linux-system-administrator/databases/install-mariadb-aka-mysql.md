# Install & Configure MariaDB / MySQL

## Task

> We need to setup a database server on Nautilus DB Server in Stratos Datacenter. Please perform the below given steps on DB Server:
>
> * Install/Configure MariaDB server.
> * Create a database named `kodekloud_db3`.
> * Create a user called `kodekloud_aim` and set its password to `BruCStnMT5`.
> * Grant full permissions to user `kodekloud_aim` on database `kodekloud_db3`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* MySQL
  * https://www.baeldung.com/linux/shell-verify-mysql-db-exists
  * https://stackoverflow.com/questions/5131931/how-to-connect-to-mysql-from-the-command-line
  * https://stackoverflow.com/questions/2428416/how-to-create-a-database-from-shell-command-in-mysql
  * https://stackoverflow.com/questions/28111674/trying-to-check-if-a-mysql--user=ser-exists-in-a-bash-script
  * https://dev.mysql.com/doc/refman/8.0/en/create--user=ser.html
  * https://dev.mysql.com/doc/refman/8.0/en/grant.html
  * https://dev.mysql.com/doc/refman/8.0/en/show-tables.html
  * https://dev.mysql.com/doc/refman/8.0/en/mysqlshow.html
  * https://stackoverflow.com/questions/28111674/trying-to-check-if-a-mysql--user=ser-exists-in-a-bash-script
  * https://stackoverflow.com/questions/8829102/check-if-mysql-table-exists-without--user=sing-select-from-syntax


## Steps

```bash
# Connect to database server
ssh peter@stdb01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Change to root
sudo -i

# Install useful tools, e.g. ss, nslookup, clear, and required packages
dnf install -y iproute bind-utils ncurses bash-completion mariadb-server

# Enable and start mariadb
systemctl enable --now mariadb

# Set up the database
export DB_NAME=kodekloud_db3
export DB_USER=kodekloud_aim
export DB_PASSWORD=BruCStnMT5

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
| kodekloud_db3 |
+---------------+
+-------------------------+
| USER()                  |
+-------------------------+
| kodekloud_aim@localhost |
+-------------------------+
```

We are done.
