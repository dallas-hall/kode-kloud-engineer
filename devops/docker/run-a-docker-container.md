# Run A Docker Container

## Task

> Nautilus DevOps team is testing some applications deployment on some of the application servers. They need to deploy a `nginx` container on Application Server 3. Please complete the task as per details given below:<br><br>On Application Server 3 create a container named `nginx_3` using image `nginx` with `alpine` tag and make sure container is in running state.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `docker -h`

## Steps


```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Switch to root
sudo -i

# Run container
docker run -d --name nginx_3 nginx:alpine
```

```
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
4db1b89c0bd1: Pull complete
bd338968799f: Pull complete
6a107772494d: Pull complete
9f05b0cc5f6e: Pull complete
4c5efdb87c4a: Pull complete
c8794a7158bf: Pull complete
8de2a93581dc: Pull complete
768e67c521a9: Pull complete
Digest: sha256:2d194184b067db3598771b4cf326cfe6ad5051937ba1132b8b7d4b0184e0d0a6
Status: Downloaded newer image for nginx:alpine
6bd125029d3b281094186e349914c09ebe9150edbabeda8d6d2d7c89313de700
```


```bash
# Check container is running
docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
6bd125029d3b        nginx:alpine        "/docker-entrypoint.…"   22 seconds ago      Up 18 seconds       80/tcp              nginx_3
```

We are done.
