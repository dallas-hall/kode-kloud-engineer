# Installing Postfix Mail & Dovecot

## Task

> xFusionCorp Industries has planned to set up a common email server in Stork DC. After several meetings and recommendations they have decided to use `postfix` as their mail transfer agent and `dovecot` as an IMAP/POP3 server. We would like you to perform the following steps:
>
> 1. Install and configure `postfix` on Stork DC mail server.
> 2. Create an email account `james@stratos.xfusioncorp.com` identified by `8FmzjvFU6S`.
> 3. Set its mail directory to `/home/james/Maildir`.
> 4. Install and configure `dovecot` on the same server.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Change postfix hostname
  * https://www.postfix.org/postconf.5.html#myhostname
* Change postfix domain
  * https://www.postfix.org/postconf.5.html#mydomain
* Change postfix mailbox path
  * https://www.postfix.org/postconf.5.html#home_mailbox
* What is Dovecot
  * https://en.wikipedia.org/wiki/Dovecot_(software)
* Configure Dovecot for IMAP and POP3
  * https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/deploying_different_types_of_servers/assembly_installing-and-configuring-dovecot-for-imap-and-pop3_deploying-different-types-of-servers
  * https://www.nbtechsupport.co.in/2021/08/linux-postfix-mail-server.html
* View mail with Dovecot
  * https://superuser.com/questions/1523594/how-can-i-view-mail-using-doveadm

## Steps

```bash
# Connect to the first app server
ssh groot@stmail01

# Switch to root
sudo -i

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Install postfix
dnf install -y postfix

# Enable and start postfix
systemctl enable --now postfix
```

```
Created symlink /etc/systemd/system/multi-user.target.wants/postfix.service → /usr/lib/systemd/system/postfix.service.
Job for postfix.service failed because the control process exited with error code.
See "systemctl status postfix.service" and "journalctl -xe" for details.
```

```bash
# Take a backup before changes
cp /etc/postfix/main.cf /etc/postfix/main.cf.old
```

Follow the troubleshooting at [troubleshooting-postfix.md](troubleshooting-postfix.md) as it was the same error.

```bash
# Default postfix hostname is fully qualified domain name. So no changes.
# Default postfix domain is FQDN minus the first component. So no changes.

# Set postfix allowed email address
egrep -i '^#?mydestination' /etc/postfix/main.cf
```

```
mydestination = $myhostname, localhost.$mydomain, localhost
#mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
#mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain,
```

```bash
# Comment out default
sed -i -r 's/mydestination = \$myhostname, localhost.\$mydomain, localhost$/#mydestination = \$myhostname, localhost.\$mydomain, localhost/g' /etc/postfix/main.cf

# Uncomment line with $mydomain
sed -i -r 's/#mydestination = \$myhostname, localhost.\$mydomain, localhost, \$mydomain$/mydestination = \$myhostname, localhost.\$mydomain, localhost, \$mydomain/g' /etc/postfix/main.cf

# Change mailbox dir from default /var/lib/mail/$USER
grep -i home_mailbox /etc/postfix/main.cf
```

```
#home_mailbox = Mailbox
#home_mailbox = Maildir/
```

```bash
sed -i -r 's/^#home_mailbox = Maildir/home_mailbox = Maildir/g' /etc/postfix/main.cf

# Restart the service
systemctl restart postfix

# Create the user
useradd james

# Change james's password
passwd james

# Install dovecot
dnf install -y dovecot

# Enable dovecot
systemctl enable --now dovecot

# Take a backup
cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf.old

# Configure
vi /etc/dovecot/dovecot.conf
```

```
# Protocols we want to be serving.
#protocols = imap pop3 lmtp
protocols = imap pop3
```

```bash
# Restart the service
systemctl restart dovecot

# Confirm dovecot is running
doveadm instance list
```

```
path                      name    last used           running
/run/dovecot              dovecot 2023-09-28 08:53:14 yes
```

```bash
# Send a test mail
echo "Subject: Test" | sendmail james@localhost
```

```bash
# Check the mail for the user.
doveadm fetch -u james "text" MAILBOX INBOX UNSEEN
```

```
text:
Return-Path: <root@stmail01.stratos.xfusioncorp.com>
X-Original-To: james@localhost
Delivered-To: james@localhost
Received: by stmail01.stratos.xfusioncorp.com (Postfix, from userid 0)
        id 3F6424A1349E; Fri, 29 Sep 2023 00:41:23 +0000 (UTC)
Subject: Test
Message-Id: <20230929004123.3F6424A1349E@stmail01.stratos.xfusioncorp.com>
Date: Fri, 29 Sep 2023 00:41:23 +0000 (UTC)
From: root <root@stmail01.stratos.xfusioncorp.com>
```

We are done.
