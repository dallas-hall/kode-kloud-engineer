# Install SELinux

## Task

> The xFusionCorp Industries security team recently did a security audit of their infrastructure and came up with ideas to improve the application and server security. They decided to use SElinux for an additional security layer. They are still planning how they will implement it; however, they have decided to start testing with app servers, so based on the recommendations they have the following requirements:<br><br>Install the required packages of SElinux on *App server 2* in *Stratos Datacenter* and disable it permanently for now; it will be enabled after making some required configuration changes on this host. Don't worry about rebooting the server as there is already a reboot scheduled for tonight's maintenance window. Also ignore the status of SElinux command line right now; the final status after reboot should be `disabled`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How to install and configure SELinux in CentOS 7 - https://hostingultraso.com/help/centos/installing-configuring-important-selinux-tools-centos & https://www.cyberciti.biz/faq/disable-selinux-on-centos-7-rhel-7-fedora-linux/

## Steps

```bash
# Connect to the first app server
ssh steve@stapp02

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
CentOS Linux release 7.6
```

```bash
# Check current SELinux status
sestatus
```

```
-bash: sestatus: command not found
```

```bash
# Install SELinux
yum install -y selinux-policy setroubleshoot-server policycoreutils libselinux setools-console
```

```
...
Installed:
  policycoreutils.x86_64 0:2.5-34.el7
  selinux-policy.noarch 0:3.13.1-268.el7_9.2
  setools-console.x86_64 0:3.3.8-4.el7
  setroubleshoot-server.x86_64 0:3.2.30-8.el7
...
Complete!
```

```bash
# Check current SELinux status
sestatus
```

```
SELinux status:                 disabled
```

```bash
# Check current SELinux mode.
cat /etc/selinux/config
```

```
...
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
...
```

```bash
# Disable SELinux & check config update.
sed -i -r 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
cat /etc/selinux/config
```

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```