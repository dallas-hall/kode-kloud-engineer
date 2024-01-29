# Docker Compose Application Deployment

## Task

> The Nautilus Application development team recently finished development of one of the apps that they want to deploy on a containerized platform. The Nautilus Application development and DevOps teams met to discuss some of the basic pre-requisites and requirements to complete the deployment. The team wants to test the deployment on one of the app servers before going live and set up a complete containerized stack using a docker compose fie. Below are the details of the task:
>
> On App Server 1 in Stratos Datacenter create a docker compose file `/opt/data/docker-compose.yml` (should be named exactly).
>
> The compose should deploy two services (web and DB), and each service should deploy a container as per details below:
>
> For web service:
> * Container name must be `php_blog`.
> * Use image `php`with any `apache` tag.
> * Map `php_blog` container's port `80` with host port `6400`
> * Map `php_blog` container's `/var/www/html` volume with host volume `/var/www/html`.
>
> For DB service:
>
> * Container name must be `mysql_blog`.
> * Use image `mariadb` with any tag (preferably `latest`).
> * Map `mysql_blog` container's port `3306` with host port `3306`.
> * Map `mysql_blog` container's `/var/lib/mysql` volume with host volume `/var/lib/mysql`.
> * Set `MYSQL_DATABASE=database_blog`and use any custom user ( except `root` ) with some complex password for DB connections.

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
* php Docker images
  * https://hub.docker.com/_/php?tab=tags/
* MariaDB Docker images
  * https://hub.docker.com/_/mariadb?tab=tags/

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
cat /etc/*rel*

# Switch to root
sudo -i

# Go to the working path.
cd /opt/data

# Create the Docker Compose file - https://docs.docker.com/compose/
cat > /opt/data/docker-compose.yml
```

```yaml
version: '3.3'
services:
  php_blog:
    container_name: php_blog
    # https://docs.docker.com/config/containers/container-networking/
    ports:
      - "6400:80" # host:container
    volumes:
      - /var/www/html:/var/www/html # host:container
    image: php:apache
  mysql_blog:
    container_name: mysql_blog
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    image: mariadb:latest
    # https://hub.docker.com/_/mariadb
    environment:
      MYSQL_DATABASE: database_blog
      MYSQL_ROOT_PASSWORD: Abc123!@#
      MYSQL_USER: php_blog
      MYSQL_PASSWORD: Abc123!@#
```

```bash
# Build the images and deploy the containers in detached mode.
docker-compose up -d

# Check the images
docker image ls
```

```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
php          apache    c2ddac79b604   9 days ago     507MB
mariadb      latest    2b54778e06a3   2 months ago   404MB
```

```bash
# Check the running containers.
docker ps -a
```

```
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                    NAMES
2f1f0a89d7f4   mariadb:latest   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:3306->3306/tcp   mysql_blog
1b6bcdf8fab8   php:apache       "docker-php-entrypoi…"   2 minutes ago   Up 2 minutes   0.0.0.0:6400->80/tcp     php_blog
```

```bash
# Test the apache container connection, it was okay.
curl localhost:6400
```

```html
<html>
    <head>
        <title>Welcome to xFusionCorp Industries!</title>
    </head>

    <body>
        Welcome to xFusionCorp Industries!    </body>
</html>
```

We saw the content of `/var/www/html/index.php`. We are done.
