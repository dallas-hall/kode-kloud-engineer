# Troubleshooting Postfix Mail

## Task

> Some users of the monitoring app have reported issues with xFusionCorp Industries mail server. They have a mail server in Stork DC where they are using `postfix` mail transfer agent. Postfix service seems to fail. Try to identify the root cause and fix it.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* After investigating the problem, search for "postfix fatal: parameter inet_interfaces: no local interface found for ::1" - https://nixhive.com/fatal-parameter-inet_interfaces-no-local-interface-found-for-1/ & https://askavy.com/postfix-error-fatal-parameter-inet_interfaces-no-local-interface-found-for-1/

## Steps

```bash
# Connect to the first app server
ssh groot@stmail01

# Switch to root
sudo -i

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Check postfix status
systemctl status postfix
```

```
● postfix.service - Postfix Mail Transport Agent
   Loaded: loaded (/usr/lib/systemd/system/postfix.service; disabled; vendor preset: disabled)
   Active: inactive (dead)

Sep 29 00:56:51 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Collecting.
Sep 29 00:56:56 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Collecting.
Sep 29 00:57:13 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Collecting.
```

The service isn't started so lets try that.

```bash
systemctl start postfix
```

```
Job for postfix.service failed because the control process exited with error code.
See "systemctl status postfix.service" and "journalctl -xe" for details.
```

Lets check the logs.

```bash
systemctl status postfix -l
```

```
● postfix.service - Postfix Mail Transport Agent
   Loaded: loaded (/usr/lib/systemd/system/postfix.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Fri 2023-09-29 00:57:52 UTC; 2s ago
  Process: 839 ExecStart=/usr/sbin/postfix start (code=exited, status=1/FAILURE)
  Process: 838 ExecStartPre=/usr/libexec/postfix/chroot-update (code=exited, status=0/SUCCESS)
  Process: 823 ExecStartPre=/usr/libexec/postfix/aliasesdb (code=exited, status=75)
  Process: 809 ExecStartPre=/usr/sbin/restorecon -R /var/spool/postfix/pid/master.pid (code=exited, status=0/SUCCESS)

Sep 29 00:57:51 stmail01.stratos.xfusioncorp.com postfix[839]: warning: /etc/postfix/main.cf, line 135: overriding earli
er entry: inet_interfaces=all
Sep 29 00:57:51 stmail01.stratos.xfusioncorp.com postfix[839]: fatal: parameter inet_interfaces: no local interface foun
d for ::1
Sep 29 00:57:52 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Child 839 belongs to postfix.service.
Sep 29 00:57:52 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Control process exited, code=exited status
=1
Sep 29 00:57:52 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Got final SIGCHLD for state start.
Sep 29 00:57:52 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Failed with result 'exit-code'.
Sep 29 00:57:52 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Changed start -> failed
Sep 29 00:57:52 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Job postfix.service/start finished, result
=failed
Sep 29 00:57:52 stmail01.stratos.xfusioncorp.com systemd[1]: Failed to start Postfix Mail Transport Agent.
Sep 29 00:57:52 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Unit entered failed state.
```

The problem to search the internet for is: `fatal: parameter inet_interfaces: no local interface found for ::1`. I followed logic at https://askavy.com/postfix-error-fatal-parameter-inet_interfaces-no-local-interface-found-for-1/


```bash
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

Implemented the fix from https://askavy.com/postfix-error-fatal-parameter-inet_interfaces-no-local-interface-found-for-1/

```bash
sed -i -r 's/inet_interfaces = localhost/inet_interfaces = 127.0.0.1/g' /etc/postfix/main.cf

# Restart and check the postfix service
systemctl restart postfix
systemctl status postfix
```

```
● postfix.service - Postfix Mail Transport Agent
   Loaded: loaded (/usr/lib/systemd/system/postfix.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-09-29 01:00:02 UTC; 13s ago
  Process: 934 ExecStart=/usr/sbin/postfix start (code=exited, status=0/SUCCESS)
  Process: 933 ExecStartPre=/usr/libexec/postfix/chroot-update (code=exited, status=0/SUCCESS)
  Process: 911 ExecStartPre=/usr/libexec/postfix/aliasesdb (code=exited, status=0/SUCCESS)
  Process: 897 ExecStartPre=/usr/sbin/restorecon -R /var/spool/postfix/pid/master.pid (code=exited, status=0/SUCCESS)
 Main PID: 1197 (master)
    Tasks: 3 (limit: 1340692)
   Memory: 17.1M
   CGroup: /docker/64e954e2bea5de0e1cd5b9feb1e0ca1fd6ce9db579958ec6ae85532345ff71a0/system.slice/postfix.service
           ├─1197 /usr/libexec/postfix/master -w
           ├─1198 pickup -l -t unix -u
           └─1199 qmgr -l -t unix -u

Sep 29 01:00:02 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Control process exited, code=exited status=0
Sep 29 01:00:02 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Got final SIGCHLD for state start.
Sep 29 01:00:02 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: New main PID 1197 belongs to service, we are happy.
Sep 29 01:00:02 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Main PID loaded: 1197
Sep 29 01:00:02 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Changed start -> running
Sep 29 01:00:02 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Job postfix.service/start finished, result=done
Sep 29 01:00:02 stmail01.stratos.xfusioncorp.com systemd[1]: Started Postfix Mail Transport Agent.
Sep 29 01:00:02 stmail01.stratos.xfusioncorp.com systemd[1]: postfix.service: Failed to send unit change signal for postfix.service: Connection reset by peer
Sep 29 01:00:02 stmail01.stratos.xfusioncorp.com postfix/pickup[1198]: warning: /etc/postfix/main.cf, line 135: overriding earlier entry: inet_interfaces=all
Sep 29 01:00:02 stmail01.stratos.xfusioncorp.com postfix/qmgr[1199]: warning: /etc/postfix/main.cf, line 135: overriding earlier entry: inet_interfaces=all
```

```bash
# Send a test mesasge with available client
echo "Subject: Test" | sendmail root@localhost

# Check test message
cat /var/mail/root
```

```
From root@stmail01.stratos.xfusioncorp.com  Fri Sep 29 01:00:39 2023
Return-Path: <root@stmail01.stratos.xfusioncorp.com>
X-Original-To: root@localhost
Delivered-To: root@localhost
Received: by stmail01.stratos.xfusioncorp.com (Postfix, from userid 0)
        id 6517613F0C33; Fri, 29 Sep 2023 01:00:39 +0000 (UTC)
Subject: Test
Message-Id: <20230929010039.6517613F0C33@stmail01.stratos.xfusioncorp.com>
Date: Fri, 29 Sep 2023 01:00:39 +0000 (UTC)
From: root <root@stmail01.stratos.xfusioncorp.com>
Status: RO
```

We are done.
