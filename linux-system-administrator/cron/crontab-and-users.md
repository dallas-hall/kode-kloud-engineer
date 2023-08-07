# Crontab And Users

## Task

> To stick with the security compliances, the Nautilus project team has decided to apply some restrictions on crontab access so that only allowed users can create/update the cron jobs. Limit crontab access to below specified users on App Server 3.<br><br>Allow `crontab` access to `james` user and deny the same to `garrett` user.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `crontab` access restriction.
  * https://www.digitalocean.com/community/tutorials/how-to-use-cron-to-automate-tasks-centos-8

## Steps

```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Create the allow list
cat > /etc/cron.allow
```

```
root
james
```

```bash
# Test the access for james
sudo -u james crontab -l
```

```
no crontab for james
```

```bash
# Test the access for james
sudo -u garret crontab -l
```

```
You (garrett) are not allowed to use this program (crontab)
See crontab(1) for more information
```

We are done.
