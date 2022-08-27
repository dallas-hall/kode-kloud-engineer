# Disable Root SSH Login

## Task

> After doing some security audits of servers, xFusionCorp Industries security team has implemented some new security policies. One of them is to disable direct root login through SSH.<br><br>Disable direct SSH root login on all app servers in Stratos Datacenter.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How to disable root login via ssh - https://www.a2hosting.com/kb/getting-started-guide/accessing-your-account/disabling-ssh-logins-for-root#CentOS-and-Fedora

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Connect to stapp01
ssh tony@stapp01

# Get root
sudo -i

# Update sshd
sed -i -r 's/#PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config

# Restart sshd to read the update
systemctl reload sshd

# Exit to thor jump host and try to ssh as root
exit

ssh root@stapp01
```

```
root@stapp01's password:
Permission denied, please try again.
```

```bash
# Repeat the sed commands for all other app servers
ssh steve@stapp02
ssh banner@stapp03
```
