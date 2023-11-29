# Docker Volume Mapping

## Task

> The Nautilus DevOps team is testing applications containerization, which issupposed to be migrated on docker container-based environments soon. In today's stand-up meeting one of the team members has been assigned a task to create and test a docker container with certain requirements. Below are more details:
>
> * On App Server 2 in Stratos DC pull `nginx` image (preferably `latest` tag but others should work too).
> * Create a new container with name `blog` from the image you just pulled.
> * Map the host volume `/opt/security` with container volume `/tmp`. There is an `sample.txt` file present on same server under `/tmp`; copy that file to `/opt/security`. Also please keep the container in running state.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Docker volume.
  * https://docs.docker.com/storage/volumes/
* Docker bind mount.
  * https://docs.docker.com/storage/bind-mounts/
  * https://stackoverflow.com/a/49952217

## Steps


```bash
# Connect to application servers
ssh steve@stapp02

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Copy the file listed above.
cp /tmp/sample.txt /opt/security/

# View the Docker Volumes, there were none.
docker volume ls

# Create the container
docker run -d --name blog --mount type=bind,source=/opt/security,target=/tmp  nginx:latest
```

```
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
1f7ce2fa46ab: Pull complete 
9b16c94bb686: Pull complete 
9a59d19f9c5b: Pull complete 
9ea27b074f71: Pull complete 
c6edf33e2524: Pull complete 
84b1ff10387b: Pull complete 
517357831967: Pull complete 
Digest: sha256:10d1f5b58f74683ad34eb29287e07dab1e90f10af243f151bb50aa5dbb4d62ee
Status: Downloaded newer image for nginx:latest
c033f13efcb5b5c3dc6240686bffa031edd6313cd1d9505c20e4020b6b999687
```

```bash
# View the new container
docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
c033f13efcb5        nginx:latest        "/docker-entrypoint.…"   21 seconds ago      Up 17 seconds       80/tcp              blog
```

We are done.
