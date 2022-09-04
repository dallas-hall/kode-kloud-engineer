# Bash Backup Script

## Task

> The production support team of xFusionCorp Industries is working on developing some bash scripts to automate different day to day tasks. One is to create a bash script for taking websites backup. They have a static website running on App Server 2 in Stratos Datacenter, and they need to create a bash script named `beta_backup.sh` which should accomplish the following tasks. (Also remember to place the script under `/scripts` directory on App Server 2)<br><br>a. Create a zip archive named `xfusioncorp_beta.zip` of `/var/www/html/beta` directory.<br><br>b. Save the archive in `/backup/` on App Server 2. This is a temporary storage, as backups from this location will be clean on weekly basis. Therefore, we also need to save this backup archive on Nautilus Backup Server.<br><br>c. Copy the created archive to Nautilus Backup Server server in `/backup/` location.<br><br>d. Please make sure script won't ask for password while copying the archive file. Additionally, the respective server user (for example, tony in case of App Server 1) must be able to run it.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None

## Steps

```bash
# Connect to application server
ssh steve@stapp02

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Create the script
cat > /scripts/beta_backup.sh
```

```bash
#!/usr/bin/env bash

echo "[INFO] Delete old backup."
rm -rfv /backup/xfusioncorp_beta.zip
echo "[INFO] Create backup."
zip /backup/xfusioncorp_beta.zip /var/www/html/beta
echo "[INFO] Check backup."
ls -Ahl --color /backup
unzip -l /backup/xfusioncorp_beta.zip
echo "[INFO] Copy backup to stbkp01."
scp /backup/xfusioncorp_beta.zip stbkp01:/backup/
echo "[INFO] Check copied backup on stbkp01."
ssh stbkp01 ls -Ahl --color /backup
```

Close the file with control + D i.e `^D`

```bash
# Make executable
chmod 755 /scripts/beta_backup.sh

# Make it runnable by steve
chown steve: /scripts/beta_backup.sh

# Set up passwordless ssh between steve@stapp02 and stbkp01
sudo -iu steve
```

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for the user `clint`.

```bash
# Run the script as steve
/scripts/beta_backup.sh
```
