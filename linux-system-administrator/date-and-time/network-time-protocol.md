# Network Time Protocol (NTP)

## Task

> The system admin team of xFusionCorp Industries has noticed an issue with some servers in *Stratos Datacenter* where some of the servers are not in sync w.r.t time. Because of this, several application functionalities have been impacted. To fix this issue the team has started using common/standard NTP servers. They are finished with most of the servers except *App Server 2*. Therefore, perform the following tasks on this server:<br><br>Install and configure NTP server on *App Server 2*.<br><br>Add NTP server `server 1.my.pool.ntp.org` in NTP configuration on *App Server 2*.<br><br>Please do not try to start/restart/stop ntp service, as we already have a restart for this service scheduled for tonight and we don't want these changes to be applied right now.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How to install and configure NTP in CentOS 7 - https://www.cyberithub.com/how-to-install-configure-ntp-server-in-rhel-centos-7-8/

## Steps

```bash
# Connect to the first app server
ssh steve@stapp02

# Get root
sudo -i

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Change current date and time settings
timedatectl
```

```
      Local time: Sun 2023-08-20 05:45:51 UTC
  Universal time: Sun 2023-08-20 05:45:51 UTC
        RTC time: n/a
       Time zone: UTC (UTC, +0000)
     NTP enabled: n/a
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

```bash
# Install ntp
yum install ntp -y
```

```
...
Installed:
  ntp.x86_64 0:4.2.6p5-29.el7.centos.2
...
Complete!
```

```bash
# Create a backup
cp -a /etc/ntp.conf /etc/ntp.conf.bak

# Check current setting
egrep '^server 1' /etc/ntp.conf
```

```
server 1.centos.pool.ntp.org iburst
```

```bash
# Configure NTP
sed -i -r 's/^server 1.*/server 1.my.pool.ntp.org iburst/g' /etc/ntp.conf
egrep '^server 1' /etc/ntp.conf
```

```
server 1.my.pool.ntp.org iburst
```

We are done.
