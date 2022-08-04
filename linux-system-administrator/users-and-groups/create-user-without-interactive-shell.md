# User Without Login Shell

## Task

> The System admin team of `xFusionCorp Industries` has installed a backup agent tool on all app servers. As per the tool's requirements they need to create a user with a non-interactive shell.<br><br>Therefore, create a user named `rose` with a non-interactive shell on the `App Server 3`

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How to create a user without a login shell - https://linoxide.com/how-to-disable-shell-access-to-user-account-in-linux/

## Steps


```bash
# Connect to the third app server
ssh banner@stapp03

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Check for existing user, there wasn't one.
grep rose /etc/passwd

# Create the user without an interactive shell
useradd -s /sbin/nologin rose

# Check the new user
grep rose /etc/passwd
```

```
rose:x:1002:1002::/home/rose:/sbin/nologin
```