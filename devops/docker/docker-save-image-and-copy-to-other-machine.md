# Docker Save Image & Copy To Another Machine

## Task

> One of the DevOps team members was working on to create a new custom docker image on App Server 1 in Stratos DC. He is done with his changes and image is saved on same server with name `beta:datacenter`. Recently a requirement has been raised by a team to use that image for testing, but the team wants to test the same on App Server 3. So we need to provide them that image on App Server 3 in Stratos DC.
>
> * On App Server 1 save the image `beta:datacenter` in an archive.
> * Transfer the image archive to App Server 3.
> * Load that image archive on App Server 3 with same name and tag which was used on App Server 3 with same name and tag which was used on App Server 1.
>
> **Note:** Docker is already installed on both servers; however, if its service is down please make sure to start it.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Save Docker image to tar file
  * https://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-using-a-repository
* Recreate the Docker image from the tar file
  * https://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-using-a-repository

## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

ssh tony@stapp01

# Check if Docker daemon is running, it was.
systemctl status docker

# Check Docker images
docker image ls
```

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
beta                datacenter          69e7a5490ee7        6 minutes ago       124MB
ubuntu              latest              e4c58958181a        7 weeks ago         77.8MB
```

```bash
# Save Docker image to tar file.
docker save -o image.tar beta:datacenter

# Zip up Docker image to save space
gzip image.tar

# Copy to the other server
scp image.tar.gz banner@stapp03:/home/banner/

# Connect to the other server, from thor
ssh banner@stapp03

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check if Docker daemon is running, it was.
systemctl status docker

# Unzip the file
mv ~banner/image.tar.gz .
gunzip image.tar.gz

# Recreate the Docker image from the tar file
docker load -i ./image.tar
```

```
256d88da4185: Loading layer [==================================================>]  80.35MB/80.35MB
95af36ab0242: Loading layer [==================================================>]  46.33MB/46.33MB
Loaded image: beta:datacenter
```

```bash
# Check it was created
docker image ls
```

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
beta                datacenter          69e7a5490ee7        16 minutes ago      124M
```

We are done.
