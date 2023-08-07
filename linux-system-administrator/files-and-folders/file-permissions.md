# File Permissions

## Task

> There are new requirements to automate a backup process that was performed manually by the xFusionCorp Industries system admins team earlier. To automate this task, the team has developed a new bash script `xfusioncorp.sh`. They have already copied the script on all required servers, however they did not make it executable on one the app server i.e App Server 3 in Stratos Datacenter.<br><br>Please give executable permissions to `/tmp/xfusioncorp.sh` script on App Server 3. Also make sure every user can execute it.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How many app servers in Stratos Datacenter? 3: stapp01, stapp02, and stapp03.
* How to change a timezone in Linux - https://linuxize.com/post/how-to-set-or-change-timezone-in-linux/

## Steps

```bash
# Connect to the first app server
ssh tony@stapp01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check file permissions
ls -Ahl --color /tmp/xfusioncorp.sh
```

```
---------- 1 root root 40 Aug  5 23:31 /tmp/xfusioncorp.sh
```

```bash
# Update file permissions & check changes
chmod 755 /tmp/xfusioncorp.sh
ls -Ahl --color /tmp/xfusioncorp.sh
```

```
-rwxr-xr-x 1 root root 40 Aug  5 23:31 /tmp/xfusioncorp.sh
```

We are done.
