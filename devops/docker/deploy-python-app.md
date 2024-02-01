# Deploy Python App

## Task

> A python app needed to be Dockerized, and then it needs to be deployed on App Server 1. We have already copied a `requirements.txt` file (having the app dependencies) under `/python_app/src/` directory on App Server 1. Further complete this task as per details mentioned below:
> * Create a `Dockerfile`under `/python_app` directory:
>   * Use any python image as the base image.
>   * Install the dependencies using `requirements.txt` file.
>   * Expose the port `5001`.
>   * Run the `server.py` script using `CMD`.
> * Build an image named `nautilus/python-app` using this Dockerfile.
> * Once image is built, create a container named `pythonapp_nautilus`:
>   * Map port `5001` of the container to the host port `8096`.
> * Once deployed, you can test the app using `curl` command on App Server 1.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create Dockerfile
  * https://docs.docker.com/get-started/02_our_app/
* Python
  * https://hub.docker.com/_/python
  * https://www.docker.com/blog/how-to-dockerize-your-python-applications/
* Build the Dockerfile
  * https://docs.docker.com/get-started/02_our_app/#build-and-test-your-image
* Run the container
  * https://docs.docker.com/get-started/02_our_app/#run-your-image-as-a-container


## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS Steam 8
cat /etc/*rel*

# Switch to root
sudo -i

# Switch to the working path.
cd /python_app

# Create the Dockerfile.
cat > Dockerfile
```

```dockerfile
# Use the official image as a parent image.
FROM python:latest

# Set the working directory.
WORKDIR /python_app

# Copy the files to the WORKDIR
COPY src/requirements.txt .
COPY src/server.py .

# Run the command inside your image filesystem.
RUN pip install -r requirements.txt

# Expose a networking port.
EXPOSE 5001

# Start the Node app.
CMD [ "python", "server.py" ]
```

Close the file with control + d i.e. `^D`

```bash
# Build the Dockerfile
docker build --tag nautilus/python-app .
```

```
Sending build context to Docker daemon  4.608kB
Step 1/7 : FROM python:latest
latest: Pulling from library/python
6a299ae9cfd9: Pull complete
e08e8703b2fb: Pull complete
68e92d11b04e: Pull complete
5b9fe7fef9be: Pull complete
09864a904dd0: Pull complete
f26050718d24: Pull complete
81c15c4db818: Pull complete
5f77fa18bfae: Pull complete
Digest: sha256:690924bb394da687beb5c2b6a439d556ee6b88659a0a4e0cba7c82c5df7a28d7
Status: Downloaded newer image for python:latest
 ---> 86ac1e3337a9
Step 2/7 : WORKDIR /python_app
 ---> Running in e952009d39fd
Removing intermediate container e952009d39fd
 ---> 1471a4bb86b7
Step 3/7 : COPY src/requirements.txt .
 ---> 028d1778fdb5
Step 4/7 : COPY src/server.py .
 ---> 1982980fa1dd
Step 5/7 : RUN pip install -r requirements.txt
 ---> Running in d102d1b2cc4f
Collecting flask (from -r requirements.txt (line 1))
  Obtaining dependency information for flask from https://files.pythonhosted.org/packages/bd/0e/63738e88e981ae57c23bad6c499898314a1110a4141f77d7bd929b552fb4/flask-3.0.1-py3-none-any.whl.metadata
  Downloading flask-3.0.1-py3-none-any.whl.metadata (3.6 kB)
Collecting Werkzeug>=3.0.0 (from flask->-r requirements.txt (line 1))
  Obtaining dependency information for Werkzeug>=3.0.0 from https://files.pythonhosted.org/packages/c3/fc/254c3e9b5feb89ff5b9076a23218dafbc99c96ac5941e900b71206e6313b/werkzeug-3.0.1-py3-none-any.whl.metadata
  Downloading werkzeug-3.0.1-py3-none-any.whl.metadata (4.1 kB)
Collecting Jinja2>=3.1.2 (from flask->-r requirements.txt (line 1))
  Obtaining dependency information for Jinja2>=3.1.2 from https://files.pythonhosted.org/packages/30/6d/6de6be2d02603ab56e72997708809e8a5b0fbfee080735109b40a3564843/Jinja2-3.1.3-py3-none-any.whl.metadata
  Downloading Jinja2-3.1.3-py3-none-any.whl.metadata (3.3 kB)
Collecting itsdangerous>=2.1.2 (from flask->-r requirements.txt (line 1))
  Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Collecting click>=8.1.3 (from flask->-r requirements.txt (line 1))
  Obtaining dependency information for click>=8.1.3 from https://files.pythonhosted.org/packages/00/2e/d53fa4befbf2cfa713304affc7ca780ce4fc1fd8710527771b58311a3229/click-8.1.7-py3-none-any.whl.metadata
  Downloading click-8.1.7-py3-none-any.whl.metadata (3.0 kB)
Collecting blinker>=1.6.2 (from flask->-r requirements.txt (line 1))
  Obtaining dependency information for blinker>=1.6.2 from https://files.pythonhosted.org/packages/fa/2a/7f3714cbc6356a0efec525ce7a0613d581072ed6eb53eb7b9754f33db807/blinker-1.7.0-py3-none-any.whl.metadata
  Downloading blinker-1.7.0-py3-none-any.whl.metadata (1.9 kB)
Collecting MarkupSafe>=2.0 (from Jinja2>=3.1.2->flask->-r requirements.txt (line 1))
  Obtaining dependency information for MarkupSafe>=2.0 from https://files.pythonhosted.org/packages/2c/db/e20eb0899e6cfc0bc51c21d0b62fb138f5e014443c340cacbd250768b968/MarkupSafe-2.1.4-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata
  Downloading MarkupSafe-2.1.4-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (3.0 kB)
Downloading flask-3.0.1-py3-none-any.whl (101 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 101.2/101.2 kB 1.7 MB/s eta 0:00:00
Downloading blinker-1.7.0-py3-none-any.whl (13 kB)
Downloading click-8.1.7-py3-none-any.whl (97 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 97.9/97.9 kB 6.2 MB/s eta 0:00:00
Downloading Jinja2-3.1.3-py3-none-any.whl (133 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 133.2/133.2 kB 8.4 MB/s eta 0:00:00
Downloading werkzeug-3.0.1-py3-none-any.whl (226 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 226.7/226.7 kB 24.2 MB/s eta 0:00:00
Downloading MarkupSafe-2.1.4-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (28 kB)
Installing collected packages: MarkupSafe, itsdangerous, click, blinker, Werkzeug, Jinja2, flask
Successfully installed Jinja2-3.1.3 MarkupSafe-2.1.4 Werkzeug-3.0.1 blinker-1.7.0 click-8.1.7 flask-3.0.1 itsdangerous-2.1.2
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip is available: 23.2.1 -> 23.3.2
[notice] To update, run: pip install --upgrade pip
Removing intermediate container d102d1b2cc4f
 ---> 53a7fa3e6ef9
Step 6/7 : EXPOSE 5001
 ---> Running in 905dfb437c3a
Removing intermediate container 905dfb437c3a
 ---> 5f263be07fdf
Step 7/7 : CMD [ "python", "server.py" ]
 ---> Running in 2cc4c40ef2d9
Removing intermediate container 2cc4c40ef2d9
 ---> 40dd59a1fe23
Successfully built 40dd59a1fe23
Successfully tagged nautilus/python-app:latest
```

```bash
# Check the built image
docker image ls
```

```
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
nautilus/python-app   latest              40dd59a1fe23        47 seconds ago      1.03GB
python                latest              86ac1e3337a9        6 weeks ago         1.02GB
```


```bash
# Run the container with host:container exposed ports.
docker run --publish 8096:5001 --detach --name pythonapp_nautilus nautilus/python-app

# Check container is running and the port binding is working on the CRE host. Was OK.
docker ps -a
```

```
CONTAINER ID        IMAGE                 COMMAND              CREATED              STATUS              PORTS                    NAMES
ccbd8e794c01        nautilus/python-app   "python server.py"   About a minute ago   Up 35 seconds       0.0.0.0:8096->5001/tcp   pythonapp_nautilus
```

```bash
# Check container connectivity from the CRE host. Saw a custom message returned.
curl localhost:8096
```

We are done.
