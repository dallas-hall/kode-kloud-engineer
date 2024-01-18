# Troubleshoot A Dockerfile

## Task

> The Nautilus DevOps team is working to create new images per requirements shared by the development team. One of the team members is working to create a `Dockerfile` on App Server 2 in Stratos DC. While working on it she ran into issues in which the docker build is failing and displaying errors. Look into the issue and fix it to build an image as per details mentioned below:
> * The `Dockerfile` is placed on App Server 2 under `/opt/docker` directory.
> * Fix the issues with this file and make sure it is able to build the image.
> * Do not change base image, any other valid configuration within Dockerfile, or any of the data been used — for example, `index.html`.


## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `docker -h`
* Dockerfile
  * https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
  * https://docs.docker.com/engine/reference/commandline/build/
* httpd image
  * https://hub.docker.com/_/httpd

## Steps


```bash
# Connect to application servers
ssh steve@stapp02

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Switch to root
sudo -i

# Backup the Dockerfile.
cp -a /opt/docker/Dockerfile /opt/docker/Dockerfile.old

# Try to build the Dockerfile
cd /opt/docker/
docker build -t my-apache2 .
```

```
Sending build context to Docker daemon  10.75kB
Step 1/8 : FROM httpd:2.4.43
2.4.43: Pulling from library/httpd
bf5952930446: Pull complete 
3d3fecf6569b: Pull complete 
b5fc3125d912: Pull complete 
3c61041685c0: Pull complete 
34b7e9053f76: Pull complete 
Digest: sha256:cd88fee4eab37f0d8cd04b06ef97285ca981c27b4d685f0321e65c5d4fd49357
Status: Downloaded newer image for httpd:2.4.43
 ---> f1455599cc2e
Step 2/8 : RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf.d/httpd.conf
 ---> Running in c274813d8e5f
sed: can't read /usr/local/apache2/conf.d/httpd.conf: No such file or directory
The command '/bin/sh -c sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf.d/httpd.conf' returned a non-zero code: 2
```


```bash
# Fix the current error.
vi /opt/docker/Dockerfile
```
Change from

```dockerfile
FROM httpd:2.4.43

RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf.d/httpd.conf

RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf.d/httpd.conf

RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf.d/httpd.conf

RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf.d/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt

COPY certs/server.key /usr/local/apache2/conf/server.key

COPY html/index.html /usr/local/apache2/htdocs/
```

to 

```dockerfile
FROM httpd:2.4.43

RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf

RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' /usr/local/apache2/conf/httpd.conf

RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' /usr/local/apache2/conf/httpd.conf

RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' /usr/local/apache2/conf/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt

COPY certs/server.key /usr/local/apache2/conf/server.key

COPY html/index.html /usr/local/apache2/htdocs/
```

and close the file `:x`

```bash
# Try to build the Dockerfile
docker build -t my-apache2 .
```

```
Sending build context to Docker daemon  10.75kB
Step 1/8 : FROM httpd:2.4.43
 ---> f1455599cc2e
Step 2/8 : RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
 ---> Using cache
 ---> 45e262bc1336
Step 3/8 : RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' /usr/local/apache2/conf/httpd.conf
 ---> Using cache
 ---> d60c3c60f7b3
Step 4/8 : RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' /usr/local/apache2/conf/httpd.conf
 ---> Using cache
 ---> 4b49dc4964ac
Step 5/8 : RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' /usr/local/apache2/conf/httpd.conf
 ---> Using cache
 ---> 66d0f3edf927
Step 6/8 : COPY certs/server.crt /usr/local/apache2/conf/server.crt
 ---> Using cache
 ---> 486806925ea5
Step 7/8 : COPY certs/server.key /usr/local/apache2/conf/server.key
 ---> Using cache
 ---> be653db8c350
Step 8/8 : COPY html/index.html /usr/local/apache2/htdocs/
 ---> Using cache
 ---> 81f9312af177
Successfully built 81f9312af177
Successfully tagged my-apache2:latest
```

```bash
# Try to run the image
docker run -dit --name my-running-app -p 8080:80 my-apache2

# View the running cointainers
docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                  NAMES
3e192d6fb14c        my-apache2          "httpd-foreground"       9 seconds ago       Up 5 seconds               0.0.0.0:8080->80/tcp   my-running-app
```

We are done.
