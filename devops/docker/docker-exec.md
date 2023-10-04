# Docker Exec Into Container

## Task

> One of the Nautilus DevOps team members was working to configure services on a kkloud container that is running on *App Server 1* in Stratos Datacenter. Due to some personal work he is on PTO for the rest of the week, but we need to finish his pending work ASAP. Please complete the remaining work as per details given below:
>
> a. Install `apache2` in `kkloud` container using apt that is running on *App Server 1* in Stratos Datacenter.
> b. Configure Apache to listen on port `5001` instead of default http port. Do not bind it to listen on specific IP or hostname only, i.e it should listen on `localhost`, `127.0.0.1`, `container ip`, etc.
> c. Make sure Apache service is up and running inside the container. Keep the container in running state at the end.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Ubuntu 18 LTS
  * https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-18-04

## Steps


```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check running containers.
docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b1fb18c75f6f        ubuntu:18.04        "/bin/bash"         5 minutes ago       Up 5 minutes                            kkloud
```

```bash
# Connect to the container
docker exec -it b1fb18c75f6f /bin/sh

# Check current Linux version, it was Ubuntu 18 LTS
 cat /etc/*rel*

# Install apache2 and vim
apt install apache2 vim

# Find the file to edit
grep -ir 80 /etc/apache2/
```

```
/etc/apache2/ports.conf:Listen 80
```

```bash
# Create a backup.
cp /etc/apache2/ports.conf /etc/apache2/ports.conf.old

# Update the config
vi /etc/apache2/apache2.conf
```

Change `Listen 80` to `Listen 5001` and press `:x` to save and exit.

```bash
# Start the service
service start apache2

# Install iproute so we can see port bindings.
apt install iproute -y
ss -lntp
```

```
State               Recv-Q               Send-Q                                Local Address:Port                               Peer Address:Port
LISTEN              0                    511                                         0.0.0.0:5001                                    0.0.0.0:*                   users:(("apache2",pid=3744,fd=3),("apache2",pid=3743,fd=3),("apache2",pid=3740,fd=3))
```

Exit out of the container with `exit`. We are done.
