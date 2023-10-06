# Troubleshooting MariaDB

## Task

> There is a critical issue going on with the Nautilus application in Stratos DC. The production support team identified that the application is unable to connect to the database. After digging into the issue, the team found that mariadb service is down on the database server.Look into the issue and fix it.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None.

## Steps

```bash
# Connect to the database server
ssh peter@stdb01

# Switch to root
sudo -i

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Check the mariadb service
systemctl status mariadb
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
● mariadb.service - MariaDB 10.3 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since Fri 2023-10-06 07:53:56 UTC; 1min 42s ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
  Process: 775 ExecStartPost=/usr/libexec/mysql-check-upgrade (code=exited, status=0/SUCCESS)
  Process: 732 ExecStart=/usr/libexec/mysqld --basedir=/usr $MYSQLD_OPTS $_WSREP_NEW_CLUSTER (code=exited, status=0/SUCCESS)
  Process: 567 ExecStartPre=/usr/libexec/mysql-prepare-db-dir mariadb.service (code=exited, status=0/SUCCESS)
  Process: 487 ExecStartPre=/usr/libexec/mysql-check-socket (code=exited, status=0/SUCCESS)
 Main PID: 732 (code=exited, status=0/SUCCESS)
   Status: "MariaDB server is down"

Oct 06 07:53:55 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Got notification message from PID 732 (STATUS=ensuring dirty
 buffer pool are written to log, EXTEND_TIMEOUT_USEC=30000000)
Oct 06 07:53:55 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Got notification message from PID 732 (STATUS=Free innodb bu
ffer pool, EXTEND_TIMEOUT_USEC=30000000)
Oct 06 07:53:56 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Got notification message from PID 732 (STATUS=MariaDB server
 is down)
Oct 06 07:53:56 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Got notification message from PID 732 (STATUS=MariaDB server
 is down)
Oct 06 07:53:56 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Child 732 belongs to mariadb.service.
Oct 06 07:53:56 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Main process exited, code=exited, status=0/SUCCESS
Oct 06 07:53:56 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Succeeded.
Oct 06 07:53:56 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Changed stop-sigterm -> dead
Oct 06 07:53:56 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Job mariadb.service/stop finished, result=done
Oct 06 07:53:56 stdb01.stratos.xfusioncorp.com systemd[1]: Stopped MariaDB 10.3 database server.
```

</details>

```bash
# Try to start the mariadb service
systemctl start mariadb
```

```
Job for mariadb.service failed because the control process exited with error code.
See "systemctl status mariadb.service" and "journalctl -xe" for details.
```

```bash
# Check the journal logs for the mariadb service
journalctl -u mariadb.service
```

```
...
Oct 06 07:55:59 stdb01.stratos.xfusioncorp.com systemd[1018]: PR_SET_MM_ARG_START failed, proceeding without: Operation not permitted
...
```

Is an interesting line, lets check the app logs.

```bash
# Check the mariadb app logs found in the journal logs
cat /var/log/mariadb/mariadb.log
```

```
...
d on IP: '::'.
2023-10-06  7:55:59 0 [ERROR] mysqld: Can't create/write to file '/run/mariadb/mariadb.pid'
...
```

```bash
# Check the permission denied error on the file
ls -Ahl /run/mariadb/mariadb.pid
```

```
ls: cannot access '/run/mariadb/mariadb.pid': No such file or directory
```

```bash
# Check the permission denied error on the folder
ls -Adhl /run/mariadb/
```

```
drwxr-xr-x 2 root mysql 40 Oct  6 07:53 /run/mariadb/
```

```bash
# Fix the folder owner, could also have changed the permissions for the group.
chown mysql: /run/mariadb/

# Start the mariadb service & check it
systemctl start mariadb
systemctl status mariadb
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
● mariadb.service - MariaDB 10.3 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-10-06 08:02:24 UTC; 19s ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
  Process: 1382 ExecStartPost=/usr/libexec/mysql-check-upgrade (code=exited, status=0/SUCCESS)
  Process: 1277 ExecStartPre=/usr/libexec/mysql-prepare-db-dir mariadb.service (code=exited, status=0/SUCCESS)
  Process: 1231 ExecStartPre=/usr/libexec/mysql-check-socket (code=exited, status=0/SUCCESS)
 Main PID: 1339 (mysqld)
   Status: "Taking your SQL requests now..."
    Tasks: 30 (limit: 1340692)
   Memory: 79.6M
   CGroup: /docker/4b0be86d16a41421a216f7707b2df9634df404774cefd37b056e16eb12f76828/system.slice/mariadb.service
           └─1339 /usr/libexec/mysqld --basedir=/usr

Oct 06 08:02:24 stdb01.stratos.xfusioncorp.com systemd[1382]: Applying namespace mount on /run/systemd/unit-root/var/tmp
Oct 06 08:02:24 stdb01.stratos.xfusioncorp.com systemd[1382]: Successfully mounted /var/tmp/systemd-private-de5c20b73d1944f0b6f500b1f955
3652-mariadb.service-zpcC8z/tmp to /run/systemd/unit-root/var/tmp
Oct 06 08:02:24 stdb01.stratos.xfusioncorp.com systemd[1382]: mariadb.service: Executing: /usr/libexec/mysql-check-upgrade
Oct 06 08:02:24 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Child 1382 belongs to mariadb.service.
Oct 06 08:02:24 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Control process exited, code=exited status=0
Oct 06 08:02:24 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Got final SIGCHLD for state start-post.
Oct 06 08:02:24 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Changed start-post -> running
Oct 06 08:02:24 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Job mariadb.service/start finished, result=done
Oct 06 08:02:24 stdb01.stratos.xfusioncorp.com systemd[1]: Started MariaDB 10.3 database server.
Oct 06 08:02:24 stdb01.stratos.xfusioncorp.com systemd[1]: mariadb.service: Failed to send unit change signal for mariadb.service: Conne
ction reset by peer
```

</details>

We are done.
