# Create A Scheduled Jenkins Job For Database Backups

## Task

> There is a requirement to create a Jenkins job to automate the database backup. Below you can find more details to accomplish this task:
>
> Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username `admin` and `Adm!n321` password.
>
> * Create a Jenkins job named `database-backup`.
> * Configure it to take a database dump of the `kodekloud_db01` database present on the Database server in Stratos Datacenter, the database user is `kodekloud_roy` and password is `asdfgdsd`.
> * The dump should be named in `db_$(date +%F).sql` format, where date +%F is the current date.
> * Copy the `db_$(date +%F).sql` dump to the Backup Server under location `/home/clint/db_backups`.
> * Further, schedule this job to run periodically at `*/10 * * * *` (please use this exact schedule format).
>
> **Note:** Once you restart Jenkins service it will go down for some time so please wait for the Jenkins login page to come back before clicking on the Check button.  For these scenarios requiring changes to be done in a web UI, please take screenshots so you can share them with us for review in case your task is marked incomplete. You may also consider using screen recording software like loom.com to record and share your work

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create a job
  * https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm
* `sshpass`
  * https://stackoverflow.com/a/16734873
* `sudo -S`
  * https://superuser.com/a/67766
* MySQL command line connection
  * https://stackoverflow.com/questions/5131931/how-to-connect-to-mysql-from-the-command-line
* MySQL backups
  * https://dev.mysql.com/doc/refman/8.0/en/using-mysqldump.html
  * https://dev.mysql.com/doc/refman/8.0/en/mysqldump-sql-format.html
  * https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html

## Steps

* Click the Jenkins button to access to UI through a broswer.
* Login with the credentials.
* Follow https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm to create the job listed above.
  * Add the following shell script.

```bash
# Variables
export DB_NAME='kodekloud_db01'
export DB_USER='kodekloud_roy'
export DB_PASSWORD='asdfgdsd'
export DB_BACKUP_FILE="db_$(date +%F).sql"
export DB_BACKUP_PATH='/home/clint/db_backups'
export DB_SERVER_USER='peter'
export DB_SERVER_PASSWORD='Sp!dy'
export DB_SERVER='stdb01.stratos.xfusioncorp.com'
export BACKUP_SERVER_USER='clint'
export BACKUP_SERVER_PASSWORD='H@wk3y3'
export BACKUP_SERVER='stbkp01.stratos.xfusioncorp.com'

# Connect to the database server and create the backup
sshpass -p "$DB_SERVER_PASSWORD" ssh -o StrictHostKeyChecking=no "$DB_SERVER_USER"@"$DB_SERVER" "mysqldump --databases $DB_NAME --user=$DB_USER --password=$DB_PASSWORD > /tmp/$DB_BACKUP_FILE && ls -Ahl /tmp/$DB_BACKUP_FILE && md5sum /tmp/$DB_BACKUP_FILE"

# Copy the backup from the DB server to the Jenkins server
sshpass -p "$DB_SERVER_PASSWORD" scp -o StrictHostKeyChecking=no -r "$DB_SERVER_USER@$DB_SERVER:/tmp/$DB_BACKUP_FILE" "/tmp/$DB_BACKUP_FILE"

sshpass -p "$DB_SERVER_PASSWORD" ssh -o StrictHostKeyChecking=no "$DB_SERVER_USER"@"$DB_SERVER" "ls -Ahl /tmp/$DB_BACKUP_FILE && md5sum /tmp/$DB_BACKUP_FILE"

# Copy the backup from the Jenkins server to the backup server.
sshpass -p "$BACKUP_SERVER_PASSWORD" scp -o StrictHostKeyChecking=no -r "/tmp/$DB_BACKUP_FILE" "$BACKUP_SERVER_USER@$BACKUP_SERVER:$DB_BACKUP_PATH"

sshpass -p "$BACKUP_SERVER_PASSWORD" ssh -o StrictHostKeyChecking=no "$BACKUP_SERVER_USER"@"$BACKUP_SERVER" "ls -Ahl $DB_BACKUP_PATH && md5sum $DB_BACKUP_PATH/$DB_BACKUP_FILE"
```

* Trigger the job.

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/database-backup
[database-backup] $ /bin/sh -xe /tmp/jenkins11479287812925407720.sh
+ export DB_NAME=kodekloud_db01
+ export DB_USER=kodekloud_roy
+ export DB_PASSWORD=asdfgdsd
+ date +%F
+ export DB_BACKUP_FILE=db_2023-11-22.sql
+ export DB_BACKUP_PATH=/home/clint/db_backups
+ export DB_SERVER_USER=peter
+ export DB_SERVER_PASSWORD=Sp!dy
+ export DB_SERVER=stdb01.stratos.xfusioncorp.com
+ export BACKUP_SERVER_USER=clint
+ export BACKUP_SERVER_PASSWORD=H@wk3y3
+ export BACKUP_SERVER=stbkp01.stratos.xfusioncorp.com
+ sshpass -p Sp!dy ssh -o StrictHostKeyChecking=no peter@stdb01.stratos.xfusioncorp.com mysqldump --databases kodekloud_db01 --user=kodekloud_roy --password=asdfgdsd > /tmp/db_2023-11-22.sql && ls -Ahl /tmp/db_2023-11-22.sql && md5sum /tmp/db_2023-11-22.sql
-rw-rw-r-- 1 peter peter 46K Nov 22 01:54 /tmp/db_2023-11-22.sql
b3d94b156d9ecffc0e4f4d8814f0a4b0  /tmp/db_2023-11-22.sql
+ sshpass -p Sp!dy scp -o StrictHostKeyChecking=no -r peter@stdb01.stratos.xfusioncorp.com:/tmp/db_2023-11-22.sql /tmp/db_2023-11-22.sql
+ sshpass -p Sp!dy ssh -o StrictHostKeyChecking=no peter@stdb01.stratos.xfusioncorp.com ls -Ahl /tmp/db_2023-11-22.sql && md5sum /tmp/db_2023-11-22.sql
-rw-rw-r-- 1 peter peter 46K Nov 22 01:54 /tmp/db_2023-11-22.sql
b3d94b156d9ecffc0e4f4d8814f0a4b0  /tmp/db_2023-11-22.sql
+ sshpass -p H@wk3y3 scp -o StrictHostKeyChecking=no -r /tmp/db_2023-11-22.sql clint@stbkp01.stratos.xfusioncorp.com:/home/clint/db_backups
+ sshpass -p H@wk3y3 ssh -o StrictHostKeyChecking=no clint@stbkp01.stratos.xfusioncorp.com ls -Ahl /home/clint/db_backups && md5sum /home/clint/db_backups/db_2023-11-22.sql
total 48K
-rw-r--r-- 1 clint clint 46K Nov 22 01:54 db_2023-11-22.sql
b3d94b156d9ecffc0e4f4d8814f0a4b0  /home/clint/db_backups/db_2023-11-22.sql
Finished: SUCCESS
```

</details>

We are done.
