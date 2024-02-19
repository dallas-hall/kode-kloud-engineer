# Bash Database Backup Script

## Task

> The Nautilus DevOps team is working on to develop a bash script to automate some tasks. As per the requirements shared with the team database related tasks needed to be automated. Below you can find more details about the same:
>
> * Write a bash script named `/opt/scripts/database.sh` on Database Server. The `mariadb` database server is already installed on this server.
> * Add code in the script to perform some database related operations as per conditions given below:
>   * Create a new database named `kodekloud_db01`. If this database already exists on the server then script should print a message `Database already exists` and if the database does not exist then create the same and script should print `Database kodekloud_db01 has been created`. Further, create a user named `kodekloud_roy` and set its password to `asdfgdsd`, also give full access to this user on newly created database (remember to use wildcard host while creating the user).
> * Now check if the database (if it was already there) already contains some data (tables)if so then script should print `database is not empty` otherwise import the database dump `/opt/db_backups/db.sql` and print `imported database dump into kodekloud_db01 database`.
> * Take a mysql dump which should be named as `kodekloud_db01.sql` and save it under `/opt/db_backups/` directory.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Bash
  * https://stackoverflow.com/questions/26675681/how-to-check-the-exit-status--user=sing-an-if-statement
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
# Connect to the database server
ssh peter@stdb01

# Switch to root
sudo -i

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Create the backup script
cat > /opt/scripts/database.sh
```

```bash
#!/usr/bin/env bash
export DB_NAME=kodekloud_db01
export DB_USER=kodekloud_roy
export DB_PASSWORD=asdfgdsd
export DB_BACKUP_SCRIPT=/opt/db_backups/db.sql
export DB_BACKUP_OUTPUT_FILE=kodekloud_db01.sql

if mysql -e "use $DB_NAME"; then
    echo "Database already exists"
else
    echo "Database kodekloud_db01 has been created"
    mysql -e "create database $DB_NAME;"
fi

mysql -e "CREATE USER IF NOT EXISTS '$DB_USER' IDENTIFIED BY '$DB_PASSWORD';"
mysql -e "GRANT ALL ON $DB_NAME.* TO '$DB_USER';"
mysql --user="$DB_USER" --password="$DB_PASSWORD" --database="$DB_NAME" -e 'SELECT VERSION(); SELECT DATABASE(); SELECT USER();'

mysqlshow "$DB_NAME;"
SQL_RESULT=$(mysql --user="$DB_USER" --password="$DB_PASSWORD" --database="$DB_NAME" -e "SELECT count(*) FROM information_schema.TABLES WHERE TABLE_SCHEMA = '$DB_NAME';")

if [[ "$SQL_RESULT" -ge 1 ]]; then
    echo "database is not empty"
else
    echo "imported database dump into $DB_NAME database."
    chmod 755 "$DB_BACKUP_SCRIPT"
    "$DB_BACKUP_SCRIPT"
fi

echo "Dumping database."
mysqldump --databases $DB_NAME --user=$DB_USER --password=$DB_PASSWORD > "/opt/db_backups/$DB_BACKUP_OUTPUT_FILE"
```

```bash
# Execute the script.
chmod 755 /opt/scripts/database.sh
/opt/scripts/database.sh
```

```
-- Dump completed on 2024-02-19  6:21:44
[root@stdb01 scripts]# ./database.sh
Database already exists
+-----------------+
| VERSION()       |
+-----------------+
| 10.3.28-MariaDB |
+-----------------+
+----------------+
| DATABASE()     |
+----------------+
| kodekloud_db01 |
+----------------+
+-------------------------+
| USER()                  |
+-------------------------+
| kodekloud_roy@localhost |
+-------------------------+
Wildcard: kodekloud_db01;
+-----------+
| Databases |
+-----------+
+-----------+
imported database dump into kodekloud_db01 database.
Dumping database.
```

We are done.
