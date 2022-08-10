# Install SELinux

## Task

> There is a critical issue going on with the Nautilus application in Stratos DC. The production support team identified that the application is unable to connect to the database. After digging into the issue, the team found that mariadb service is down on the database server.<br><br>Look into the issue and fix it.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None.

## Steps

```bash
# Connect to the first app server
ssh peter@stdb01

# Get root
sudo -i

# Check Linux version
cat /etc/*release*
```

```
CentOS Linux release 7.6.1810 (Core)
Derived from Red Hat Enterprise Linux 7.6 (Source)
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

CentOS Linux release 7.6.1810 (Core)
CentOS Linux release 7.6.1810 (Core)
cpe:/o:centos:centos:7
```

```bash
# Check the mariadb service
systemctl status mariadb
```

```
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
Aug 10 09:42:35 stdb01.stratos.xfusioncorp.com systemd[1]: Collecting mariadb.service
```

```bash
# Try to start the mariadb service
systemctl start mariadb
```

```
Job for mariadb.service failed because the control process exited with error code. See "systemctl status mariadb.service" and "journalctl -xe" for details.
```

```bash
# Check the journal logs for the mariadb service
journalctl -u mariadb.service
```

```
...
Aug 10 09:42:44 stdb01.stratos.xfusioncorp.com mysqld_safe[707]: 220810 09:42:44 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'
...
```

```bash
# Check the mariadb app logs found in the journal logs
cat /var/log/mariadb/mariadb.log
```

```
...
220810  9:42:46 [ERROR] mysqld: Can't create/write to file '/var/run/mariadb/mariadb.pid' (Errcode: 13)
220810  9:42:46 [ERROR] Can't start server: can't create PID file: Permission denied
220810 09:42:46 mysqld_safe mysqld from pid file /var/run/mariadb/mariadb.pid ended
...
```

```bash
# Check the permission denied error on the file
ls -Ahl /var/run/mariadb/mariadb.pid
```

```
ls: cannot access /var/run/mariadb/mariadb.pid: No such file or directory
```

```bash
# Check the permission denied error on the folder
ls -Adhl /var/run/mariadb/
```

```
drwxr-xr-x 2 root mysql 40 Aug 10 09:40 /var/run/mariadb/
```

```bash
# Fix the folder owner, could also have changed the permissions for the group.
chown mysql: /var/run/mariadb/

# Start the mariadb service & check it
systemctl start mariadb
systemctl status mariadb
```

```
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-08-10 09:50:33 UTC; 11s ago
  Process: 1008 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
  Process: 973 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 1007 (mysqld_safe)
   CGroup: /docker/f3b03bb3a5d7f51f7fd7ef6a14515df166c3b8faf2368953a401902fc169ec9f/system.slice/mariadb.service
           ├─1007 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─1171 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/...

Aug 10 09:50:31 stdb01.stratos.xfusioncorp.com systemd[1007]: Executing: /usr/bin/mysqld_safe --basedir=/usr
Aug 10 09:50:31 stdb01.stratos.xfusioncorp.com systemd[1008]: Executing: /usr/libexec/mariadb-wait-ready 1007
Aug 10 09:50:31 stdb01.stratos.xfusioncorp.com mysqld_safe[1007]: 220810 09:50:31 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'.
Aug 10 09:50:31 stdb01.stratos.xfusioncorp.com mysqld_safe[1007]: 220810 09:50:31 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
Aug 10 09:50:33 stdb01.stratos.xfusioncorp.com systemd[1]: Child 1008 belongs to mariadb.service
Aug 10 09:50:33 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: control process exited, code=exited status=0
Aug 10 09:50:33 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service got final SIGCHLD for state start-post
Aug 10 09:50:33 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service changed start-post -> running
Aug 10 09:50:33 stdb01.stratos.xfusioncorp.com systemd[1]: Job mariadb.service/start finished, result=done
Aug 10 09:50:33 stdb01.stratos.xfusioncorp.com systemd[1]: Started MariaDB database server.
```
