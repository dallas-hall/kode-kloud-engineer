# Deploy Node App

## Task

> There is a requirement to Dockerize a Node app and to deploy the same on App Server 3. Under `/node_app` directory on App Server 3, we have already placed a `package.json` file that describes the app dependencies and `server.js` file that defines a web app framework.
>
> * Create a `Dockerfile` (name is case sensitive) under `/node_app` directory:
>   * Use any node image as the base image.
>   * Install the dependencies using `package.json` file.
>   * Use `server.js` in the `CMD`.
>   * Expose port `8082`.
> * The build image should be named as `nautilus/node-web-app`.
> * Now run a container named `nodeapp_nautilus` using this image.
>   * Map the container port `8082` with the host port `8091`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create Dockerfile
  * https://docs.docker.com/get-started/02_our_app/
* Build Node App
  * https://hub.docker.com/_/node/
  * https://github.com/nodejs/docker-node
  * https://www.digitalocean.com/community/tutorials/how-to-build-a-node-js-application-with-docker
  * https://docs.npmjs.com/cli/v8/commands/npm-install
* Build the Dockerfile
  * https://docs.docker.com/get-started/02_our_app/#build-and-test-your-image
* Run the container
  * https://docs.docker.com/get-started/02_our_app/#run-your-image-as-a-container


## Steps

```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Switch to root
sudo -i

# Switch to the working path.
cd /node_app

# Create the Dockerfile.
cat > Dockerfile
```

```dockerfile
# Use the official image as a parent image.
FROM node:21

# Set the working directory.
WORKDIR /node_app

# Copy the files to the WORKDIR
COPY package.json .
COPY server.js .

# Run the command inside your image filesystem.
RUN npm install

# Expose a networking port.
EXPOSE 8082

# Start the Node app.
CMD [ "node", "server.js" ]
```

Close the file with control + d i.e. `^D`

```bash
# Build the Dockerfile
docker build --tag nautilus/node-web-app .
```

```
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM node:21
21: Pulling from library/node
1b13d4e1a46e: Pull complete
1c74526957fc: Pull complete
30d855997954: Pull complete
ad5739181616: Pull complete
b2aed33d4664: Pull complete
1688fca40fa1: Pull complete
e028f1255c74: Pull complete
49b4ed55fa73: Pull complete
Digest: sha256:abc4a25c8b5a2b460f3144aabfc8941ecd7e4fb721e0b14b635e70394c1899fb
Status: Downloaded newer image for node:21
 ---> 80eddb4b8564
Step 2/7 : WORKDIR /node_app
 ---> Running in 7525ee78659a
Removing intermediate container 7525ee78659a
 ---> 92bc35a3c193
Step 3/7 : COPY package.json .
 ---> e798d44694bd
Step 4/7 : COPY server.js .
 ---> c77a0150970d
Step 5/7 : RUN npm install
 ---> Running in eccb6f5df761

added 62 packages, and audited 63 packages in 3s

11 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
npm notice
npm notice New minor version of npm available! 10.2.4 -> 10.4.0
npm notice Changelog: <https://github.com/npm/cli/releases/tag/v10.4.0>
npm notice Run `npm install -g npm@10.4.0` to update!
npm notice
Removing intermediate container eccb6f5df761
 ---> 82184a951063
Step 6/7 : EXPOSE 8082
 ---> Running in 309760cf047b
Removing intermediate container 309760cf047b
 ---> ed0a18dd8826
Step 7/7 : CMD [ "node", "server.js" ]
 ---> Running in 5352671be646
Removing intermediate container 5352671be646
 ---> db98b1586429
Successfully built db98b1586429
Successfully tagged nautilus/node-web-app:latest
```

```bash
# Check the built image
docker image ls
```

```
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
nautilus/node-web-app   latest              db98b1586429        59 seconds ago      1.11GB
node                    21                  80eddb4b8564        13 hours ago        1.1GB
```


```bash
# Run the container with host:container exposed ports.
docker run --publish 8091:8082 --detach --name nodeapp_nautilus nautilus/node-web-app

# Check container is running and the port binding is working on the CRE host. Was OK.
docker ps -a
```

```
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                    NAMES
c7434a9243c2        nautilus/node-web-app   "docker-entrypoint.s…"   5 minutes ago       Up 4 minutes        0.0.0.0:8091->8082/tcp   nodeapp_nautilus
```

```bash
# Check container connectivity from the CRE host. Saw the dafault Apache httpd html.
curl localhost:8091
```

We are done.
