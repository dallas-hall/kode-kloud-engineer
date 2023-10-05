# Create A Docker File

## Task

> As per recent requirements shared by the Nautilus application development team, they need custom images created for one of their projects. Several of the initial testing requirements are already been shared with DevOps team. Therefore, create a docker file `/opt/docker/Dockerfile` (please keep D capital of Dockerfile) on App server 3 in Stratos DC and configure to build an image with the following requirements:
>
> a. Use `ubuntu` as the base image.
> b. Install `apache2` and configure it to work on `3004` port. (do not update any other Apache configuration settings like document root etc).

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create Dockerfile
  * https://docs.docker.com/get-started/02_our_app/
* Build the Dockerfile
  * https://docs.docker.com/get-started/02_our_app/#build-and-test-your-image
* Run the container
  * https://docs.docker.com/get-started/02_our_app/#run-your-image-as-a-container
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

# There weren't any running containers.
docker ps -a

# Create the Dockerfile.
cat > /opt/docker/Dockerfile
```

```dockerfile
# Use the official image as a parent image.
# https://github.com/dhanugupta/docker-apache-ubuntu/blob/master/Dockerfile
FROM ubuntu

# Run the command inside your image filesystem.
RUN apt-get -y update && apt-get -y upgrade
# Disable tzdata interactive questions - https://serverfault.com/a/992421
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install apache2

# Manually set up the apache environment variables
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid

# Add metadata to the image to describe which port the container is listening on at runtime.
EXPOSE 3004

# Update Apache listneing port
RUN echo "Listen 3004" >> /etc/apache2/apache2.conf

# Start apache.
CMD /usr/sbin/apache2ctl -D FOREGROUND
```

Close the file with control + d i.e. `^D`

```bash
# Build the Dockerfile
docker build --tag my-apache2 /opt/docker

# Run the container.
docker run --publish 3004:3004 --detach --name my-apache2-container my-apache2

# Check container is running and the port binding is working on the CRE host. Was OK.
docker ps -a
ss -lntp

# Check container connectivity from the CRE host. Saw the dafault Apache httpd html.
curl localhost:3004
```

We are done.
