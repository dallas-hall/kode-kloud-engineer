# Postfix Mail

## Task

> Some users of the monitoring app have reported issues with xFusionCorp Industries mail server. They have a mail server in Stork DC where they are using `postfix` mail transfer agent. Postfix service seems to fail. Try to identify the root cause and fix it.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Where is the mail server? stmail01.stratos.xfusioncorp.com
* After investigating the problem, search for "postfix fatal: parameter inet_interfaces: no local interface found for ::1" - https://nixhive.com/fatal-parameter-inet_interfaces-no-local-interface-found-for-1/ & https://askavy.com/postfix-error-fatal-parameter-inet_interfaces-no-local-interface-found-for-1/

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
# Check postfix status
systemctl status postfix
```

```
● postfix.service - Postfix Mail Transport Agent
   Loaded: loaded (/usr/lib/systemd/system/postfix.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Sun 2022-08-07 03:26:48 UTC; 9min ago
  Process: 490 ExecStart=/usr/sbin/postfix start (code=exited, status=1/FAILURE)
  Process: 489 ExecStartPre=/usr/libexec/postfix/chroot-update (code=exited, status=0/SUCCESS)
  Process: 450 ExecStartPre=/usr/libexec/postfix/aliasesdb (code=exited, status=75)

Aug 07 03:26:47 stmail01.stratos.xfusioncorp.com postfix[490]: fatal: parameter inet_interfaces: no local interface found for ::1
Aug 07 03:26:48 stmail01.stratos.xfusioncorp.com systemd[1]: Child 490 belongs to postfix.service
Aug 07 03:26:48 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: control process exited, code=exited status=1
Aug 07 03:26:48 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service got final SIGCHLD for state start
Aug 07 03:26:48 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service changed start -> failed
Aug 07 03:26:48 stmail01.stratos.xfusioncorp.com systemd[1]: Job postfix.service/start finished, result=failed
Aug 07 03:26:48 stmail01.stratos.xfusioncorp.com systemd[1]: Failed to start Postfix Mail Transport Agent.
Aug 07 03:26:48 stmail01.stratos.xfusioncorp.com systemd[1]: Unit postfix.service entered failed state.
Aug 07 03:26:48 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service failed.
Aug 07 03:26:48 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: cgroup is empty
```

```bash
# The problem to search the internet for is: fatal: parameter inet_interfaces: no local interface found for ::1
# Followed logic at https://askavy.com/postfix-error-fatal-parameter-inet_interfaces-no-local-interface-found-for-1/

# Check postfix configuration
grep -i inet_interfaces /etc/postfix/main.cf
```

```
# The inet_interfaces parameter specifies the network interface
inet_interfaces = all
#inet_interfaces = $myhostname
#inet_interfaces = $myhostname, localhost
inet_interfaces = localhost
# the address list specified with the inet_interfaces parameter.
# receives mail on (see the inet_interfaces parameter).
# to $mydestination, $inet_interfaces or $proxy_interfaces.
# - destinations that match $inet_interfaces or $proxy_interfaces,
# unknown@[$inet_interfaces] or unknown@[$proxy_interfaces] is returned
```

```bash
# Implemented the fix from https://askavy.com/postfix-error-fatal-parameter-inet_interfaces-no-local-interface-found-for-1/
sed -i -r 's/inet_interfaces = localhost/inet_interfaces = 127.0.0.1/g' /etc/postfix/main.cf

# Restart and check the postfix service
systemctl restart postfix
systemctl status postfix
```

```
● postfix.service - Postfix Mail Transport Agent
   Loaded: loaded (/usr/lib/systemd/system/postfix.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2022-08-07 03:41:10 UTC; 11s ago
  Process: 811 ExecStop=/usr/sbin/postfix stop (code=exited, status=1/FAILURE)
  Process: 841 ExecStart=/usr/sbin/postfix start (code=exited, status=0/SUCCESS)
  Process: 840 ExecStartPre=/usr/libexec/postfix/chroot-update (code=exited, status=0/SUCCESS)
  Process: 838 ExecStartPre=/usr/libexec/postfix/aliasesdb (code=exited, status=0/SUCCESS)
 Main PID: 912 (master)
   CGroup: /docker/a2fd341c2c14406ec43cc3f83828466a5f1ed1913765b25b6972694e781fc6fa/system.slice/postfix.service
           ├─912 /usr/libexec/postfix/master -w
           ├─913 pickup -l -t unix -u
           └─914 qmgr -l -t unix -u

Aug 07 03:41:09 stmail01.stratos.xfusioncorp.com systemd[841]: Executing: /usr/sbin/postfix start
Aug 07 03:41:10 stmail01.stratos.xfusioncorp.com postfix/master[912]: daemon started -- version 2.10.1, configuration /etc/postfix
Aug 07 03:41:10 stmail01.stratos.xfusioncorp.com systemd[1]: Child 841 belongs to postfix.service
Aug 07 03:41:10 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: control process exited, code=exited status=0
Aug 07 03:41:10 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service got final SIGCHLD for state start
Aug 07 03:41:10 stmail01.stratos.xfusioncorp.com systemd[1]: New main PID 912 belongs to service, we are happy.
Aug 07 03:41:10 stmail01.stratos.xfusioncorp.com systemd[1]: Main PID loaded: 912
Aug 07 03:41:10 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service changed start -> running
Aug 07 03:41:10 stmail01.stratos.xfusioncorp.com systemd[1]: Job postfix.service/start finished, result=done
Aug 07 03:41:10 stmail01.stratos.xfusioncorp.com systemd[1]: Started Postfix Mail Transport Agent.
```

```bash
# Send a test mesasge with available client
echo "Subject: Test" | sendmail root@localhost

# Check test message
cat /var/mail/root
```

```
From root@stmail01.stratos.xfusioncorp.com  Sun Aug  7 03:46:06 2022
Return-Path: <root@stmail01.stratos.xfusioncorp.com>
X-Original-To: root@localhost
Delivered-To: root@localhost.stratos.xfusioncorp.com
Received: by stmail01.stratos.xfusioncorp.com (Postfix, from userid 0)
        id B17ED27610BB; Sun,  7 Aug 2022 03:46:06 +0000 (UTC)
Subject: Test
Message-Id: <20220807034606.B17ED27610BB@stmail01.stratos.xfusioncorp.com>
Date: Sun,  7 Aug 2022 03:46:06 +0000 (UTC)
From: root@stmail01.stratos.xfusioncorp.com (root)
```