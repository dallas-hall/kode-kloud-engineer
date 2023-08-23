# Install SELinux

## Task

> The xFusionCorp Industries security team recently did a security audit of their infrastructure and came up with ideas to improve the application and server security. They decided to use SElinux for an additional security layer. They are still planning how they will implement it; however, they have decided to start testing with app servers, so based on the recommendations they have the following requirements:<br><br>Install the required packages of SElinux on *App server 3* in *Stratos Datacenter* and disable it permanently for now; it will be enabled after making some required configuration changes on this host. Don't worry about rebooting the server as there is already a reboot scheduled for tonight's maintenance window. Also ignore the status of SElinux command line right now; the final status after reboot should be `disabled`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How to install and configure SELinux in CentOS 7
  * https://hostingultraso.com/help/centos/installing-configuring-important-selinux-tools-centos
  * https://www.cyberciti.biz/faq/disable-selinux-on-centos-7-rhel-7-fedora-linux/

## Steps

```bash
# Connect to the first app server
ssh banner@stapp03

# Get root
sudo -i

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Check current SELinux status
sestatus
```

```
-bash: sestatus: command not found
```

```bash
# Install SELinux
dnf install -y selinux-policy setroubleshoot-server policycoreutils libselinux setools-console
```

```
...
Installed:
  selinux-policy-3.14.3-123.el8.noarch
  selinux-policy-minimum-3.14.3-123.el8.noarch
  setools-console-4.3.0-3.el8.x86_64
  setroubleshoot-plugins-3.3.14-1.el8.noarch
  setroubleshoot-server-3.3.26-5.el8.x86_64
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
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

```bash
# Create a backup
cp -a /etc/selinux/config /etc/selinux/config.bak

# Disable SELinux & check config update.
sed -i -r 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
grep 'SELINUX=' /etc/selinux/config
```

```
...
SELINUX=disabled
```

We are done.
