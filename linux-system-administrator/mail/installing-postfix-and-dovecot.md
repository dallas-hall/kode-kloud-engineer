# Installing Postfix Mail & Dovecot

## Task

> xFusionCorp Industries has planned to set up a common email server in Stork DC. After several meetings and recommendations they have decided to use `postfi`x as their mail transfer agent and `dovecot` as an IMAP/POP3 server. We would like you to perform the following steps:<br><br>Install and configure `postfix` on Stork DC mail server.<br>Create an email account `john@stratos.xfusioncorp.com` identified by `ksH85UJjhb.`<br>Set its mail directory to `/home/john/Maildir`.<br>Install and configure `dovecot` on the same server.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Change postfix hostname - https://www.postfix.org/postconf.5.html#myhostname
* Change postfix domain - https://www.postfix.org/postconf.5.html#mydomain
* Change postfix mailbox path - https://www.postfix.org/postconf.5.html#home_mailbox
* What is Dovecot - https://en.wikipedia.org/wiki/Dovecot_(software)
* Confugre Dovecot for IMAP and POP3 - https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/deploying_different_types_of_servers/assembly_installing-and-configuring-dovecot-for-imap-and-pop3_deploying-different-types-of-servers
* https://www.nbtechsupport.co.in/2021/08/linux-postfix-mail-server.html

## Steps

```bash
# Connect to the first app server
ssh groot@stmail01

# Switch to root
sudo -i

# Check Linux version
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
CentOS Linux release 7.6.1810 (Core)
cpe:/o:centos:centos:7
```

```bash
# Install postfix
yum install -y postfix
```

```
...
Installed:
  postfix.x86_64 2:2.10.1-9.el7
...
```

```bash
# Enable and start postfix
systemctl enable --now postfix
```

```
Job for postfix.service failed because the control process exited with error code. See "systemctl status postfix.service" and "journalctl -xe" for details.
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
...
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
...
#home_mailbox = Maildir/
```

```bash
sed -i -r 's/^#home_mailbox = Maildir/home_mailbox = Maildir/g' /etc/postfix/main.cf

# Restart the service
systemctl restart postfix

# Create the user
useradd john

# Change john's password
passwd john
```

```
Changing password for user john.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

```bash
# Install dovecot
yum install -y dovecot
```

```
...
Installed:
  dovecot.x86_64 1:2.2.36-8.el7
...
```

```bash
# Enable dovecot
systemctl enable --now dovecot
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/dovecot.service to /usr/lib/systemd/system/dovecot.service.
```

```bash
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
path                            name    last used           running
/var/run/dovecot                dovecot 2022-10-03 02:24:08 yes
```
