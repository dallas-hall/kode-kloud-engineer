# Install Docker

## Task

> Last week the Nautilus DevOps team met with the application development team and decided to containerize several of their applications. The DevOps team wants to do some testing per the following:<br><br>Install `docker-ce` and `docker-compose` packages on App Server 2.<br>Start `docker` service.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Install Docker - https://docs.docker.com/engine/install/centos/
* Install Docker Compose standalone - https://docs.docker.com/compose/install/other/

## Steps


```bash
# Connect to application servers
ssh steve@stapp02

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Switch to root
sudo su

# Set up the Docker repo
yum install -y yum-utils
```

```
...
Updated:
  yum-utils-4.0.21-23.el8.noarch

Complete!
```

```bash
# Add Docker repo
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

```
Loaded plugins: fastestmirror, ovl
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
```

```bash

# Install packages
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

```
...
Installed:
  docker-ce-3:24.0.5-1.el8.x86_64
  docker-ce-cli-1:24.0.5-1.el8.x86_64
  docker-ce-rootless-extras-24.0.5-1.el8.x86_64
  docker-compose-plugin-2.20.2-1.el8.x86_64
...
```

```bash
# Install Docker Compose manually
curl -SL https://github.com/docker/compose/releases/download/v2.20.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 42.8M  100 42.8M    0     0  12.2M      0  0:00:03  0:00:03 --:--:-- 15.3M
```

```bash
# Update permissions and create symlink
chmod 755 /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# Test Docker Compose
docker-compose --version
```

```
Docker Compose version v2.20.2
```

```bash
# Start Docker service
systemctl enable --now docker
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```

```bash
# Test Docker
systemctl status docker
```

```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-08-07 03:28:55 UTC; 37s ago
...
```

We are done.
