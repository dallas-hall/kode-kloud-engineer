# Docker Volume Mapping

## Task

> The Nautilus DevOps team is planning to host an application on a nginx-based container. There are number of tickets already been created for similar tasks. One of the tickets has been assigned to set up a nginx container on Application Server 2 in Stratos Datacenter. Please perform the task as per details mentioned below:
> * Pull `nginx:alpine-perl` docker image on Application Server 2.
> * Create a container named `beta` using the image you pulled.
> * Map host port `3003` to container port `80`. Please keep the container in running state.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Docker volume.
  * https://docs.docker.com/storage/volumes/
* Docker bind mount.
  * https://docs.docker.com/storage/bind-mounts/
  * https://stackoverflow.com/a/49952217

## Steps

```bash
# Connect to application servers
ssh steve@stapp02

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Create the container with the specified settings.
docker pull nginx:alpine

# Run the container mapping 8081 host to 80 container
docker run -d -t --name beta -p 3003:80  nginx:alpine-perl
```

```
Unable to find image 'nginx:alpine-perl' locally
alpine-perl: Pulling from library/nginx
96526aa774ef: Pull complete 
740091335c74: Pull complete 
da9c2e764c5b: Pull complete 
ade17ad21ef4: Pull complete 
4e6f462c8a69: Pull complete 
1324d9977cd2: Pull complete 
1b9b96da2c74: Pull complete 
153aef7ca07f: Pull complete 
a62fda4d68d9: Pull complete 
Digest: sha256:04821c737c095fe45b14c393d6185d080d17d86af5a7db3262ab763ae80804de
Status: Downloaded newer image for nginx:alpine-perl
93997ae450c84299349f7dfee92856367ffaa0a623be31b11f13e40db4a0e47b
```

```bash
# Check the container is running
docker ps -a 
```

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
93997ae450c8        nginx:alpine-perl   "/docker-entrypoint.…"   18 seconds ago      Up 16 seconds       0.0.0.0:3003->80/tcp   beta
```

```bash
curl localhost:3003
```

```html
<!doctype html>
<html lang="">
   <head>
      <meta charset="utf-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width,initial-scale=1">
      <link rel="icon" href="/favicon.ico">
      <title>labs-quiz-app-cli5</title>
      <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
      <script defer="defer" src="/js/chunk-vendors.fa4a391b.js"></script><script defer="defer" src="/js/app.3d19b1cd.js"></script>
      <link href="/css/chunk-vendors.157af930.css" rel="stylesheet">
      <link href="/css/app.6488a654.css" rel="stylesheet">
   </head>
   <body>
      <noscript><strong>We're sorry but labs-quiz-app-cli5 doesn't work properly without JavaScript enabled. Please enable it to continue.</strong></noscript>
      <div id="app"></div>
   </body>
</html>
```

We are done.
