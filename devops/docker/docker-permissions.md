# Docker Permissions

## Task

> One of the Nautilus project developers need access to run `docker` commands on App Server 2. This user is already created on the server. Accomplish this task as per details given below:
>
> * User `ammar` is not able to run `docker` commands on App Server 2 in Stratos DC, make the required changes so that this user can run `docker` commands without `sudo`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Sudoless Docker
  * https://docs.docker.com/engine/install/linux-postinstall/
* `usermod --help`

## Steps


```bash
# Connect to application servers
ssh steve@stapp02

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check the docker group.
grep docker /etc/group
```

```
docker:x:995:steve
```

The user isn't in that group so we will add them

```bash
# Add user to the docker group.
usermod -aG docker ammar

# Test the user's access to docker.
sudo -iu ammar

# List running containers.
docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

There are no running containers but the command worked. We are done.
