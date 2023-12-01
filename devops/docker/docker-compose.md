
# Docker Compose

## Task

> The Nautilus application development team shared static website content that needs to be hosted on the httpd web server using a containerised platform. The team has shared details with the DevOps team, and we need to set up an environment according to those guidelines. Below are the details:
>
> * On App Server 3 in Stratos DC create a container named `httpd` using a docker compose file `/opt/docker/docker-compose.yml` (please use the exact name for file).
> * Use `httpd` (preferably `latest` tag) image for container and make sure container is named as `httpd`; you can use any name for service.
> * Map `80` number port of container with port `8088` of docker host.
> * Map container's `/usr/local/apache2/htdocs` volume with `/opt/itadmin` volume of docker host which is already there. (please do not modify any data within these locations).

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Docker compose file
  * https://docs.docker.com/compose/
  * https://docs.docker.com/compose/gettingstarted/
  * https://docs.docker.com/compose/gettingstarted/#step-3-define-services-in-a-compose-file
* Docker networking
  * https://docs.docker.com/config/containers/container-networking/
* Docker container name
  * https://docs.docker.com/compose/compose-file/#container_name
* Docker detach from running container
  * https://stackoverflow.com/questions/25267372/correct-way-to-detach-from-a-container-without-stopping-it

## Steps


```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Create the Docker Compose file
cd /opt/docker
cat > docker-compose.yml
```

```yaml
version: '3.3'
services:
  httpd:
    ports:
      - "8088:80" # host:container
    volumes: 
      - /opt/itadmin:/usr/local/apache2/htdocs # host:container
    image: httpd:latest
    container_name: httpd
```

```bash
# Deploy the container
docker-compose up
```

```
Creating network "docker_default" with the default driver
Pulling httpd (httpd:latest)...
latest: Pulling from library/httpd
1f7ce2fa46ab: Pull complete
424de2a10000: Pull complete
6d9a0131505f: Pull complete
5728e491734b: Pull complete
20d3235e84ad: Pull complete
Digest: sha256:04551bc91cc03314eaab20d23609339aebe2ae694fc2e337d0afad429ec22c5a
Status: Downloaded newer image for httpd:latest
Creating httpd ... done
Attaching to httpd
httpd    | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.19.0.2. Set the 'ServerName' directive globally to suppress this message
httpd    | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.19.0.2. Set the 'ServerName' directive globally to suppress this message
httpd    | [Fri Dec 01 04:40:35.444074 2023] [mpm_event:notice] [pid 1:tid 140050041714560] AH00489: Apache/2.4.58 (Unix) configured -- resuming normal operations
httpd    | [Fri Dec 01 04:40:35.444289 2023] [core:notice] [pid 1:tid 140050041714560] AH00094: Command line: 'httpd -D FOREGROUND'
```

```bash
# Test the container
curl localhost:8088
```

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /</title>
 </head>
 <body>
<h1>Index of /</h1>
<ul><li><a href="index1.html"> index1.html</a></li>
</ul>
</body></html>
```

We are done.

