# Install & Enable A Service

## Task

> As per details shared by the development team, the new application release has some dependencies on the back end. There are some packages/services that need to be installed on all app servers under Stratos Datacenter. As per requirements please perform the following steps:
>
> a. Install `cups` package on all the application servers.
>
> b. Once installed, make sure it is enabled to start during boot.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

None.

## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS Steam 8.
cat /etc/*rel*

# Switch to root
sudo -i

# Install package
yum install -y cups
```

```
...
Installed:
...
  cups-1:2.2.6-53.el8.x86_64
...
```

```bash
# Start and enable cups
systemctl enable --now cups

# Check service status
systemctl status cups
```

```
● cups.service - CUPS Scheduler
   Loaded: loaded (/usr/lib/systemd/system/cups.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2023-09-12 08:13:10 UTC; 25s ago
...
```

```bash
# Repeat for the rest of the servers
ssh steve@stapp02
ssh banner@stapp03
```

We are done.
