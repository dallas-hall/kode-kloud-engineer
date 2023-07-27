# Create Apache User

## Task

> For security reasons the xFusionCorp Industries security team has decided to use custom Apache users for each web application hosted, rather than its default user. This will be the Apache user, so it shouldn't use the default home directory. Create the user as per requirements given below:<br><br>a. Create a user named `mark` on the App server 1 in Stratos Datacenter.<br>b. Set its UID to `1051` and home directory to `/var/www/mark`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `useradd --help`

## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Create the user
useradd -d /var/www/mark -u 1051 mark

# Check the user
grep mark /etc/passwd
```

```
mark:x:1051:1051::/var/www/mark:/bin/bash
```

We are done.