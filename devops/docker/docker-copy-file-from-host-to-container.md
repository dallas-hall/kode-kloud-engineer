# Copy Data From Docker Host To Docker Container

## Task

> The Nautilus DevOps team has some conditional data present on App Server 3 in Stratos Datacenter. There is a container ubuntu_latest running on the same server. We received a request to copy some of the data from the docker host to the container. Below are more details about the task:<br><br>On App Server 3 in Stratos Datacenter copy an encrypted file `/tmp/nautilus.txt.gpg` from docker host to `ubuntu_latest` container (running on same server) in `/usr/src/` location. Please do not try to modify this file in any way.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Docker copy files from host to container.
  * https://stackoverflow.com/a/31971697

## Steps

```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS 7.6
cat /etc/*rel*

# Switch to root
sudo -i

# View running containers
docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
bbb507528199        ubuntu              "/bin/bash"         2 minutes ago       Up 2 minutes                            ubuntu_latest
```

```bash
# Copy file from host to container
docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/usr/src/

# Check that the file copeid
docker exec ubuntu_latest ls -Ahl /usr/src/
```

```
total 4.0K
-rw-r--r-- 1 root root 105 Aug  7 07:08 nautilus.txt.gpg
```

The file is there, we are done.
