# Disable Root SSH Login

## Task

> After doing some security audits of servers, xFusionCorp Industries security team has implemented some new security policies. One of them is to disable direct root login through SSH.<br><br>Disable direct SSH root login on all app servers in Stratos Datacenter.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How to disable root login via ssh - https://www.a2hosting.com/kb/getting-started-guide/accessing-your-account/disabling-ssh-logins-for-root#CentOS-and-Fedora

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
```
