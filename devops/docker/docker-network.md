# Docker Bridge Network Setup

## Task

> The Nautilus DevOps team needs to set up several docker environments for different applications. One of the team members has been assigned a ticket where he has been asked to create some docker networks to be used later. Complete the task based on the following ticket description:<br><br>a. Create a docker network named as `news` on App Server 3 in Stratos DC.<br>b. Configure it to use `bridge` drivers.<br>c. Set it to use subnet `172.168.0.0/24` and iprange `172.168.0.3/24`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Docker network.
  * https://docs.docker.com/network/
  * https://docs.docker.com/engine/reference/commandline/network_create/

## Steps


```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo su

# View running containers
docker ps -a
```

```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

No running containers.

```bash
# Create the docker network
docker network create --driver bridge --ip-range 172.168.0.3/24 --subnet 172.168.0.0/24 news
```

```
d824ba98e8d754ea540febc2f03998cd9b1465e9315526939d1fbffddba34f1b
```

```bash
# List Docker networks
docker network ls
```

```
NETWORK ID     NAME      DRIVER    SCOPE
279df1b36fda   bridge    bridge    local
69a34ed7b6a8   host      host      local
d824ba98e8d7   news      bridge    local
2ff81dab515d   none      null      local
```

```bash
# View our network
docker network inspect news
```

```json
[
    {
        "Name": "news",
        "Id": "d824ba98e8d754ea540febc2f03998cd9b1465e9315526939d1fbffddba34f1b",
        "Created": "2022-12-20T22:11:06.028941203Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.168.0.0/24",
                    "IPRange": "172.168.0.3/24"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

We are done.
