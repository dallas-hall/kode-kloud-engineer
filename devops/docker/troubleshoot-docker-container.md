# Troubleshoot A Docker Container

## Task

> There is a static website running within a container named nautilus, this container is running on App Server 1. Suddenly, we started facing some issues with the static website on App Server 1. Look into the issue to fix the same, you can find more details below:<br><br>Container's volume `/usr/local/apache2/htdocs` is mapped with the host's volume `/var/www/html`.<br>The website should run on host port `8080` on App Server 1 i.e command `curl http://localhost:8080/` should work on App Server 1.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `docker -h`
* View mounts
  * https://stackoverflow.com/questions/30133664/how-do-you-list-volumes-in-docker-containers

## Steps


```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check running container(s)
docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS                     PORTS               NAMES
4a62d3999ca2        httpd               "httpd-foreground"   7 minutes ago       Exited (0) 7 minutes ago                       nautilus
```

```bash
# Restart to container
docker restart 4a

```bash
# Check running container(s)
docker ps -a
```

```
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                  NAMES
4a62d3999ca2        httpd               "httpd-foreground"   7 minutes ago       Up 1 second         0.0.0.0:8080->80/tcp   nautilus
```

The port binding looks fine, so lets check the mounts.

```bash
docker inspect -f '{{ json .Mounts }}' 4a
```

```json
[
  {
    "Type": "bind",
    "Source": "/var/www/html",
    "Destination": "/usr/local/apache2/htdocs",
    "Mode": "",
    "RW": true,
    "Propagation": "rprivate"
  }
]
```

The mount looks correct, so lets check the connection.

```bash
# Check connection
curl http://localhost:8080/
```

```
Welcome to xFusionCorp Industries!
```

We are done.
