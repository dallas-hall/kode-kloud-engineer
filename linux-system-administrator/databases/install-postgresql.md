# httpd Password Pkodekloud_timrotected Folder

## Task

> The Nautilus application development team has shared that they are planning to deploy one newly developed application on Nautilus infra in Stratos DC. The application uses PostgreSQL database, so as a pre-requisite we need to set up PostgreSQL database server as per requirements shared below:<br><br>a. Install and configure PostgreSQL database on Nautilus database server.<br>b. Create a database user `kodekloud_tim` and set its password to `YchZHRcLkL`.<br>c. Create a database `kodekloud_db9` and grant full permissions to user `kodekloud_tim` on this database.<br>d. Make appropriate settings to allow all local clients (local socket connections) to connect to the `kodekloud_db9` database through `kodekloud_tim` user using md5 method (Please do not try to encrypt password with md5sum).<br>e. At the end its good to test the db connection using these new credentials from `root` user or server's `sudo` user.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* CentOS 7 has 9.2.24 - https://www.postgresql.org/docs/9.2/
* Initialise the DB cluster https://www.postgresql.org/docs/9.2/creating-cluster.html
* Start the DB cluster https://www.postgresql.org/docs/9.2/server-start.html
* Create the user https://www.postgresql.org/docs/9.2/sql-createrole.html
* Create the database https://www.postgresql.org/docs/9.2/sql-createdatabase.html
* Update the authentication settings - https://www.postgresql.org/docs/9.2/auth-pg-hba-conf.html
* Reload the service - https://www.postgresql.org/docs/9.2/app-pg-ctl.html

## Steps

```bash
# Connect to database server
ssh peter@stdb01

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Change to root
sudo -i

# Install PostgreSQL
yum install -y postgresql-server
```

```
...
Installed:
  postgresql-server.x86_64 0:9.2.24-8.el7_9
...
```

```bash
# Initialise the DB cluster
sudo -iu postgres
pg_ctl initdb
```

```
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

fixing permissions on existing directory /var/lib/pgsql/data ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 32MB
creating configuration files ... ok
creating template1 database in /var/lib/pgsql/data/base/1 ... ok
initializing pg_authid ... ok
initializing dependencies ... ok
creating system views ... ok
loading system objects' descriptions ... ok
creating collations ... ok
creating conversions ... ok
creating dictionaries ... ok
setting privileges on built-in objects ... ok
creating information schema ... ok
loading PL/pgSQL server-side language ... ok
vacuuming database template1 ... ok
copying template1 to template0 ... ok
copying template1 to postgres ... ok

WARNING: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    /usr/bin/postgres -D /var/lib/pgsql/data
or
    /usr/bin/pg_ctl -D /var/lib/pgsql/data -l logfile start
```

```bash
# start the DB cluster
pg_ctl -D /var/lib/pgsql/data -l postgres_log start
```

```
server starting
```

```bash
# Connect to the DB cluster
psql
```

```
psql (9.2.24)
Type "help" for help.

postgres=#
```

```SQL
--Create our user
CREATE USER kodekloud_tim WITH PASSWORD 'YchZHRcLkL';
```

```
CREATE ROLE
```

```sql
--Create our database
CREATE DATABASE kodekloud_db9 OWNER kodekloud_tim;
```

```
CREATE DATABASE
```

```bash
# Quit the database
\q

# Update the authentication settings.
# Replace DATABASE with kodekloud_db9
# Replace USER all with kodekloud_tim
# Replace METHOD with md5
vi /var/lib/pgsql/data/pg_hba.conf
```

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
# "local" is for Unix domain socket connections only
#local   all             all                                     trust
local   kodekloud_db9    kodekloud_tim                           md5
```

```bash
# Reload the service
pg_ctl reload -D /var/lib/pgsql/data -l postgres_log
```

```
server signaled
```

```bash
# Check the connection as postgres user to the wrong DB
psql
```

```
psql: FATAL:  no pg_hba.conf entry for host "[local]", user "postgres", database "postgres", SSL off
```

```bash
# Check the connection as postgres user to the correct DB
psql -d kodekloud_db9
```

```
psql: FATAL:  no pg_hba.conf entry for host "[local]", user "postgres", database "kodekloud_db9", SSL off
```

```bash
# Check the connection as kodekloud_tim user to the correct DB with correct password
psql -d kodekloud_db9 -U kodekloud_tim
```

```
Password for user kodekloud_tim:
psql (9.2.24)
Type "help" for help.

kodekloud_db9=>
```
