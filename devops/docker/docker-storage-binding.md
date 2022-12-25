# Docker Storage Binding

## Task

> The Nautilus DevOps team is testing applications containerization, which is supposed to be migrated on docker container-based environments soon. In today's stand-up meeting one of the team members has been assigned a task to create and test a docker container with certain requirements. Below are more details:<br><br>a. On App Server 2 in Stratos DC pull `nginx` image (preferably `latest` tag but others should work too).<br>b. Create a new container with name `games` from the image you just pulled.<br>c. Map the host volume `/opt/sysops` with container volume `/home`. There is an `sample.txt` file present on same server under `/tmp`; copy that file to `/opt/sysops`. Also please keep the container in running state.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Docker storage binding - https://docs.docker.com/storage/bind-mounts/

## Steps

```bash
# Connect to application servers
ssh steve@stapp02

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# View running containers
docker ps -a

# Copy the file to the right location
cp /tmp/sample.txt /opt/sysops/

# Run a container with the requried specs
docker run -d -t \
  --name games \
  --mount type=bind,source=/opt/sysops,target=/home \
  nginx:latest
```

`docker run -d -t --name games -v /opt/sysops:/home nginx:latest` is the same as above.

```
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
3f4ca61aafcd: Pull complete 
50c68654b16f: Pull complete 
3ed295c083ec: Pull complete 
40b838968eea: Pull complete 
88d3ab68332d: Pull complete 
5f63362a3fa3: Pull complete 
Digest: sha256:0047b729188a15da49380d9506d65959cce6d40291ccfb4e039f5dc7efd33286
Status: Downloaded newer image for nginx:latest
a1236fa065ebe7f9c756eedd9dc314ee63fa6c6dead1b28a55fb769c09f08a66
```

```bash
# View running containers
docker ps -a
```

```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
a1236fa065eb   nginx:latest   "/docker-entrypoint.…"   9 seconds ago   Up 5 seconds   80/tcp    games
```

```bash
# View filesystem within the container
docker exec games ls /home
```

```
sample.txt
```

Our file is present so we are done.
