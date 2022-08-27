
# Change DNS Nameserver

## Task

> The system admins team of xFusionCorp Industries has set up some scripts on jump host that run on regular intervals and perform operations on all app servers in Stratos Datacenter. To make these scripts work properly we need to make sure the thor user on jump host has password-less SSH access to all app servers through their respective sudo users (i.e `tony` for app server 1). Based on the requirements, perform the following:<br><br>Set up a password-less authentication from user `thor` on jump host to all app servers through their respective sudo users.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* https://www.ssh.com/academy/ssh/keygen,  `man ssh-keygen`, & `man ssh-copy-id`
* https://www.ssh.com/ssh/config & https://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/

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
# Test passwordless ssh with users
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03

# Configure ssh config
cat > ~/.ssh/config
```

```
Host stapp01
	User tony
	IdentityFile ~/.ssh/id_ed25519
	PreferredAuthentications publickey

Host stapp02
	User steve
	IdentityFile ~/.ssh/id_ed25519
	PreferredAuthentications publickey

Host stapp03
	User banner
	IdentityFile ~/.ssh/id_ed25519
	PreferredAuthentications publickey
```

Close the file with `^D`

```bash
# Update ~/.ssh/config file permissions so it works.
chmod 644 ~/.ssh/config

# Test passwordless ssh without user
ssh stapp01
ssh stapp02
ssh stapp03
```
