# Install & Enable A Service

## Task

> As per details shared by the development team, the new application release has some dependencies on the back end. There are some packages/services that need to be installed on all app servers under Stratos Datacenter. As per requirements please perform the following steps:<br><br>a. Install `httpd` package on all the application servers.<br><br>b. Once installed, make sure it is enabled to start during boot.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

None.

## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Install package
yum install -y httpd
```

```
...
Installed:
  httpd.x86_64 0:2.4.6-97.el7.centos.5
...
```

```bash
# Start and enable httpd
systemctl start httpd
systemctl enable httpd
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

```bash
# Check service status
systemctl status httpd
```

```
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-08-29 10:14:21 UTC; 26s ago
     Docs: man:httpd(8)
           man:apachectl(8)
...
```

```bash
# Create a test file
cat > /var/www/html/index.html
```
Close the file with control + d, i.e. `^D`.

```
Hello world, from stapp01.
```

```bash
# Check httpd server from thor jumpbox
curl stapp01:80
```

```
Hello world, from stapp01.
```

```bash
# Repeat for the rest of the servers
ssh steve@stapp02
ssh banner@stapp03
```