# Create A Simple Cron Job

## Task

> The Nautilus system admins team has prepared scripts to automate several day-to-day tasks. They want them to be deployed on all app servers in Stratos DC on a set schedule. Before that they need to test similar functionality with a sample cron job. Therefore, perform the steps below:<br><br>a. Install `cronie` package on all Nautilus app servers and start `crond` service.<br><br>b. Add a cron `*/5 * * * * echo hello > /tmp/cron_text` for `root` user.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* https://crontab.guru/

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Connect to stapp01
ssh tony@stapp01

# Check current Linux version, it was CentOS Stream 8.
cat /etc/*rel*
```

```bash
# Switch to root
sudo -i

# Install cronie
yum install -y cronie
```

```
...
Package cronie-1.5.2-8.el8.x86_64 is already installed.
...
```

```bash
# Enable and start crond, and then check its status.
systemctl enable --now crond
systemctl start crond
systemctl status crond
```

```
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2023-08-28 09:10:57 UTC; 6min ago
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

After saving and quitting with `:x` you'll see:

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