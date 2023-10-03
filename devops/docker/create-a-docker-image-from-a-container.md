# Create A Docker Image From A Container

## Task

> One of the Nautilus developer was working to test new changes on a container. He wants to keep a backup of his changes to the container. A new request has been raised for the DevOps team to create a new image from this container. Below are more details about it:
>
> a. Create an image `media:xfusion` on Application Server 2 from a container `ubuntu_latest` that is running on same server.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create an image from a container.
  * https://kodekloud.com/blog/docker-create-image-from-container/
* `docker -h`

## Steps

```bash
# Connect to application servers
ssh steve@stapp02

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check running container(s)
docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
2cb7c1c6050d        ubuntu              "/bin/bash"         2 minutes ago       Up 2 minutes                            ubuntu_latest
```

```bash
# Create the image from the container.
docker container commit -a "KodeKloud Engineer task." -m "Create a backup for some lazy dev." ubuntu_latest media:xfusion
```

```
sha256:7a37978a461aedff10981c484e6d8576371eb265cc1b991240ef0d90c3862866
```

```bash
# Check Docker images.
docker images
```

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
media               xfusion             7a37978a461a        36 seconds ago      122MB
ubuntu              latest              3565a89d9e81        7 days ago          77.8MB
```

We are done.
