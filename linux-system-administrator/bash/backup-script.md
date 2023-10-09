# Bash Backup Script

## Task

> The production support team of xFusionCorp Industries is working on developing some bash scripts to automate different day to day tasks. One is to create a bash script for taking websites backup. They have a static website running on *App Server 1* in Stratos Datacenter, and they need to create a bash script named `ecommerce_backup.sh` which should accomplish the following tasks. (Also remember to place the script under `/scripts`` directory on *App Server 1*).
>
> a. Create a zip archive named `xfusioncorp_ecommerce.zip` of `/var/www/html/ecommerce` directory.
> b. Save the archive in `/backup/` on App Server 1. This is a temporary storage, as backups from this location will be clean on weekly basis. Therefore, we also need to save this backup archive on Nautilus Backup Server.
> c. Copy the created archive to Nautilus Backup Server server in `/backup/` location.
> d. Please make sure script won't ask for password while copying the archive file. Additionally, the respective server user (for example, `tony` in case of App Server 1) must be able to run it.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None

## Steps

```bash
# Connect to application server
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
cat /etc/*rel*

# Switch to root
sudo -i

# Create the script
cat > /scripts/ecommerce_backup.sh
```

```bash
#!/usr/bin/env bash

echo "[INFO] Delete old backup file."
rm -rfv /backup/xfusioncorp_ecommerce.zip
echo "[INFO] Create backup."
zip /backup/xfusioncorp_ecommerce.zip /var/www/html/ecommerce
echo "[INFO] Check backup file."
ls -Ahl --color /backup
unzip -l /backup/xfusioncorp_ecommerce.zip
echo "[INFO] Copy backup file to stbkp01."
scp /backup/xfusioncorp_ecommerce.zip clint@stbkp01:/backup/
echo "[INFO] Check copied backup file on stbkp01."
ssh clint@stbkp01 ls -Ahl --color /backup
```

Close the file with control + D i.e `^D`

```bash
# Make executable
chmod 755 /scripts/ecommerce_backup.sh

# Make it runnable by tony
chown tony:root /scripts/ecommerce_backup.sh

# Set up passwordless ssh between tony@stapp01 and stbkp01
sudo -iu tony
```

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for the user `clint`.

```bash
# Run the script as tony.
/scripts/ecommerce_backup.sh
```

```
[INFO] Delete old backup file.
removed '/backup/xfusioncorp_ecommerce.zip'
[INFO] Create backup.
  adding: var/www/html/ecommerce/ (stored 0%)
[INFO] Check backup file.
total 4.0K
-rw-rw-r-- 1 tony tony 196 Oct  9 08:38 xfusioncorp_ecommerce.zip
Archive:  /backup/xfusioncorp_ecommerce.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  10-09-2023 08:21   var/www/html/ecommerce/
---------                     -------
        0                     1 file
[INFO] Copy backup file to stbkp01.
xfusioncorp_ecommerce.zip                                                                                                     100%  196     1.1MB/s   00:00
[INFO] Check copied backup file on stbkp01.
total 4.0K
-rw-rw-r-- 1 clint clint 196 Oct  9 08:38 xfusioncorp_ecommerce.zip
```

We are done.
