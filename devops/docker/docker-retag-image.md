# Docker Pull & Retag Image

## Task

> Nautilus project developers are planning to start testing on a new project. As per their meeting with the DevOps team, they want to test containerized environment application features. As per details shared with DevOps team, we need to accomplish the following task:
>
> a. Pull `busybox:musl` image on App Server 2 in Stratos DC and re-tag (create new tag) this image as `busybox:blog`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None.

## Steps

```bash
# Connect to application servers
ssh steve@stapp02

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Switch to root
sudo -i

# Pull image
docker pull busybox:musl
```

```
musl: Pulling from library/busybox
d2dce026a06e: Pull complete
Digest: sha256:f553b7484625f0c73bfa3888e013e70e99ec6ae1c424ee0e8a85052bd135a28a
Status: Downloaded newer image for busybox:musl
docker.io/library/busybox:musl
```

```bash
# Get image ID
docker image ls
```

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             musl                46425a867adf        2 months ago        1.41MB
```

```bash
# Tag image
docker tag 46425a867adf busybox:blog

# Check new tag
docker image ls
```

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             blog                46425a867adf        2 months ago        1.41MB
busybox             musl                46425a867adf        2 months ago        1.41MB
```

We are done.
