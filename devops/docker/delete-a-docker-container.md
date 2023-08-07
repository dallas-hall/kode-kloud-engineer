# Run A Docker Container

## Task

> One of the Nautilus project developers created a container on App Server 2. This container was created for testing only and now we need to delete it. Accomplish this task as per details given below:<br><br>Delete a container named `kke-container`, its running on App Server 2 in Stratos DC.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

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
CONTAINER ID        IMAGE               COMMAND               CREATED              STATUS              PORTS               NAMES
0c2d54286780        busybox             "tail -f /dev/null"   About a minute ago   Up About a minute                       kke-container
```

```bash
# Stop the container
docker stop 0c2

# Delete the container
docker rm 0c2
```

We are done.
