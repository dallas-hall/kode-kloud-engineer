# Timezones

## Task

> The system admin team of xFusionCorp Industries has noticed an issue with some servers in *Stratos Datacenter* where some of the servers are not in sync w.r.t time. Because of this, several application functionalities have been impacted. To fix this issue the team has started using common/standard NTP servers. They are finished with most of the servers except *App Server 1*. Therefore, perform the following tasks on this server:<br><br>Install and configure NTP server on *App Server 1*.<br><br>Add NTP server `1.sg.pool.ntp.org` in NTP configuration on *App Server 1*.<br><br>Please do not try to start/restart/stop ntp service, as we already have a restart for this service scheduled for tonight and we don't want these changes to be applied right now.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How to install and configure NTP in CentOS 7 - https://www.cyberithub.com/how-to-install-configure-ntp-server-in-rhel-centos-7-8/

## Steps

```bash
# Connect to the first app server
ssh tony@stapp01

# Get root
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
# Change current date and time settings
timedatectl
```

```
      Local time: Mon 2022-08-08 07:31:19 UTC
  Universal time: Mon 2022-08-08 07:31:19 UTC
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
# Configure NTP
vi /etc/ntp.conf
```

```
...
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 1.sg.pool.ntp.org
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
...
```
