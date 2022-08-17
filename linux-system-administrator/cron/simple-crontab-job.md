# Disable Root SSH Login

## Task

> The Nautilus system admins team has prepared scripts to automate several day-to-day tasks. They want them to be deployed on all app servers in Stratos DC on a set schedule. Before that they need to test similar functionality with a sample cron job. Therefore, perform the steps below:<br><br>a. Install `cronie` package on all Nautilus app servers and start `crond` service.<br><br>b. Add a cron `*/5 * * * * echo hello > /tmp/cron_text` for `root` user.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* https://crontab.guru/

## Steps

```bash
# Create .ssh folder and keys
mkdir -p ~/.ssh
ssh-keygen -t ed25519
```

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/thor/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/thor/.ssh/id_ed25519.
Your public key has been saved in /home/thor/.ssh/id_ed25519.pub.
The key fingerprint is:
SHA256:18HXZAnXRpqTDbImwjdDsjP5gjv4iRBuBuKuEPaBbkg thor@jump_host.stratos.xfusioncorp.com
The key's randomart image is:
+--[ED25519 256]--+
|        . . ..oo*|
|       . = . o.@o|
|        B = = *.o|
|  .    . * * o . |
|+E..  . S o .    |
|B+.... . o       |
|o+=.. o          |
|+o . o o         |
|o.  . o          |
+----[SHA256]-----+
```

```bash
# Copy keys
ssh-copy-id -i ~/.ssh/id_ed25519 tony@stapp01
ssh-copy-id -i ~/.ssh/id_ed25519 steve@stapp02
ssh-copy-id -i ~/.ssh/id_ed25519 banner@stapp03
```

```
...
Number of key(s) added: 1
...
```

```bash
# Connect to stapp01
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
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
CentOS Linux release 7.6
```

```bash
# Switch to root
sudo -i

# Install cronie
yum install -y cronie
```

```
...
Installed:
  cronie.x86_64 0:1.4.11-24.el7_9

Dependency Installed:
  cronie-anacron.x86_64 0:1.4.11-24.el7_9
  crontabs.noarch 0:1.11-6.20121102git.el7

Complete!
```

```bash
# Enable and start crond, and then check its status.
systemctl enable crond
systemctl start crond
systemctl status crond
```

```
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2022-08-17 09:46:27 UTC; 5s ago
...
```

```bash
# Edit crontab entries, uses vim syntax.
crontab -e
```

```
# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# | .------------- hour (0 - 23)
# | | .---------- day of month (1 - 31)
# | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
# | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# | | | | |
# * * * * * command to be executed
# https://crontab.guru/every-5-minutes
*/5 * * * * echo hello > /tmp/cron_text
```

After saving and quitting you'll see:

```
no crontab for root - using an empty one
crontab: installing new crontab
```

```bash
# View crontab
crontab -l
```

```
# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# | .------------- hour (0 - 23)
# | | .---------- day of month (1 - 31)
# | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
# | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# | | | | |
# * * * * * command to be executed
# https://crontab.guru/every-5-minutes
*/5 * * * * echo hello > /tmp/cron_text
```

```bash
# Repeat on stapp02 and stapp03
ssh steve@stapp02
ssh banner@stapp03
```