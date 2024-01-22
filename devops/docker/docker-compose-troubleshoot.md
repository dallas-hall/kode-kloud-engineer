# Troubleshoot Docker Compose

## Task

> The Nautilus DevOps team is working to deploy one of the applications on App Server 2 in Stratos DC. Due to a misconfiguration in the docker compose file, the deployment is failing. We would like you to take a look into it to identify and fix the issues. More details can be found below:
> * `docker-compose.yml` file is present on App Server 1 under `/opt/docker` directory.
> * Try to run the same and make sure it works fine.
> * Please do not change the container names being used. Also, do not update or alter any other valid config settings in the compose file or any other relevant data that can cause app failure.


## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `docker-compose up -h`
* Docker Compose
  * https://docs.docker.com/compose/

## Steps

```bash
# Connect to application servers
ssh steve@stapp02

# Check current Linux version, it was CentOS 7.6
cat /etc/*rel*

# Switch to root
sudo -i

# Go to the working directory and create a backup
cd /opt/docker/
cp -a docker-compose.yml docker-compose.yml.old

# Try to build the Docker image.
docker-compose up -d
```

```
ERROR: The Compose file './docker-compose.yml' is invalid because:
Unsupported config option for services.web: 'port' (did you mean 'ports'?)
```

Used `vi` to change `port` to `ports` and try to rebuild with `docker-compose up -d` and the next error was:

```
ERROR: Service 'web' depends on service 'redis' which is undefined.
```

Used `vi` to change `redis_app` to `redis` and try to rebuild with `docker-compose up -d` and the next error was:

```
ERROR: invalid reference format
```

Used `vi` to change `image` to `build` and try to rebuild with `docker-compose up -d` and it built successfully. The full file is:

```yaml
version: '2'
services:
    web:
        build: ./app
        container_name: python
        ports:
            - "5000:5000"
        volumes:
            - ./app:/code
        depends_on:
            - redis
    redis:
        image: redis
        container_name: redis
```

```bash
# Check the built images.
docker images ls
```

```
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
docker_web   latest    cc1210d80561   About a minute ago   907MB
redis        latest    bdff4838c172   12 days ago          138MB
python       2.7       68e7be49c28c   3 years ago          902MB
```

```bash
# Check the running containers.
docker ps -a
```

```
CONTAINER ID   IMAGE        COMMAND                  CREATED          STATUS          PORTS                    NAMES
5372963abe5a   docker_web   "/bin/sh -c 'python …"   50 seconds ago   Up 11 seconds   0.0.0.0:5000->5000/tcp   python
e172c88c4999   redis        "docker-entrypoint.s…"   54 seconds ago   Up 49 seconds   6379/tcp                 redis
```

```bash
# Test the app
curl localhost:5000
```

```
This Compose/Flask demo has been viewed 1 time(s).
```

We are done.
