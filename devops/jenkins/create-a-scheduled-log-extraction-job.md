# Create A Scheduled Jenkins Job For Log Extraction

## Task

> The devops team of xFusionCorp Industries is working on to setup centralised logging management system to maintain and analyse server logs easily. Since it will take some time to implement, they wanted to gather some server logs on a regular basis. At least one of the app servers is having issues with the Apache server. The team needs Apache logs so that they can identify and troubleshoot the issues easily if they arise. So they decided to create a Jenkins job to collect logs from the server. Please create/configure a Jenkins job as per details mentioned below:
>
> Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username `admin` and `Adm!n321` password.
>
> * Create a Jenkins jobs named `copy-logs`.
> * Configure it to periodically build every 6 minutes to copy the Apache logs (both `access_log` and `error_logs`) from App Server 2 (from default logs location) to location `/usr/src/itadmin` on Storage Server.
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
* `scp`
  * https://unix.stackexchange.com/questions/232946/how-to-copy-all-files-from-a-directory-to-a-remote-directory-using-scp

## Steps

* Click the Jenkins button to access to UI through a broswer.
* Login with the credentials.
* Follow https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm to create the job listed above.
  * Add the following shell script.

```bash
# Variables
export APP2_SERVER_USER='steve'
export APP2_SERVER_PASSWORD='Am3ric@'
export APP2_SERVER='stapp02.stratos.xfusioncorp.com'
export STORAGE_SERVER_USER='natasha'
export STORAGE_SERVER_PASSWORD='Bl@kW'
export STORAGE_SERVER='ststor01.stratos.xfusioncorp.com'
export BACKUP_STAGING_PATH='/tmp/httpd_logs'
export BACKUP_SOURCE_PATH='/var/log/httpd'
export BACKUP_TARGET_PATH='/usr/src/itadmin'

rm -rf "$BACKUP_STAGING_PATH"
mkdir -p "$BACKUP_STAGING_PATH"

# Copy the httpd logs from the app 2 server to the Jenkins server
sshpass -p "$APP2_SERVER_PASSWORD" scp -o StrictHostKeyChecking=no -r "$APP2_SERVER_USER@$APP2_SERVER:$BACKUP_SOURCE_PATH/*" "$BACKUP_STAGING_PATH"
ls -Ahl "$BACKUP_STAGING_PATH"

# Copy the backup from the Jenkins server to the backup server.
sshpass -p "$STORAGE_SERVER_PASSWORD" scp -o StrictHostKeyChecking=no "$BACKUP_STAGING_PATH/access_log" "$BACKUP_STAGING_PATH/error_log" "$STORAGE_SERVER_USER@$STORAGE_SERVER:$BACKUP_TARGET_PATH"

sshpass -p "$STORAGE_SERVER_PASSWORD" ssh -o StrictHostKeyChecking=no "$STORAGE_SERVER_USER"@"$STORAGE_SERVER" "ls -Ahl $BACKUP_TARGET_PATH"
```

* Trigger the job.

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/copy-logs
[copy-logs] $ /bin/sh -xe /tmp/jenkins401874087400600496.sh
+ export APP2_SERVER_USER=steve
+ export APP2_SERVER_PASSWORD=Am3ric@
+ export APP2_SERVER=stapp02.stratos.xfusioncorp.com
+ export STORAGE_SERVER_USER=natasha
+ export STORAGE_SERVER_PASSWORD=Bl@kW
+ export STORAGE_SERVER=ststor01.stratos.xfusioncorp.com
+ export BACKUP_STAGING_PATH=/tmp/httpd_logs
+ export BACKUP_SOURCE_PATH=/var/log/httpd
+ export BACKUP_TARGET_PATH=/usr/src/itadmin
+ rm -rf /tmp/httpd_logs
+ mkdir -p /tmp/httpd_logs
+ sshpass -p Am3ric@ scp -o StrictHostKeyChecking=no -r steve@stapp02.stratos.xfusioncorp.com:/var/log/httpd/* /tmp/httpd_logs
Warning: Permanently added 'stapp02.stratos.xfusioncorp.com,172.16.238.11' (ECDSA) to the list of known hosts.
+ ls -Ahl /tmp/httpd_logs
total 4.0K
-rw-r--r-- 1 jenkins jenkins   0 Nov 23 00:34 access_log
-rw-r--r-- 1 jenkins jenkins 878 Nov 23 00:34 error_log
+ sshpass -p Bl@kW scp -o StrictHostKeyChecking=no /tmp/httpd_logs/access_log /tmp/httpd_logs/error_log natasha@ststor01.stratos.xfusioncorp.com:/usr/src/itadmin
Warning: Permanently added 'ststor01.stratos.xfusioncorp.com,172.16.238.15' (ECDSA) to the list of known hosts.
+ sshpass -p Bl@kW ssh -o StrictHostKeyChecking=no natasha@ststor01.stratos.xfusioncorp.com ls -Ahl /usr/src/itadmin
total 4.0K
-rw-r--r-- 1 natasha natasha   0 Nov 23 00:34 access_log
-rw-r--r-- 1 natasha natasha 878 Nov 23 00:34 error_log
Finished: SUCCESS
```

</details>

We are done.
