# Docker macvlan Network Setup

## Task

> The Nautilus DevOps team needs to set up several docker environments for different applications. One of the team members has been assigned a ticket where he has been asked to create some docker networks to be used later. Complete the task based on the following ticket description:
> * Create a docker network named as `official` on App Server 3 in Stratos DC.
> * Configure it to use `macvlan` drivers.
> * Set it to use subnet `10.10.1.0/24` and iprange `10.10.1.3/24`.

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

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# View running containers, there were none.
docker ps -a

# Create the docker network
docker network create --driver macvlan --ip-range 10.10.1.3/24 --subnet 10.10.1.0/24 official
```

```
5d13eb305eecdd13c9535d666819c3cdc3f7bc84f1a75e9f8cd63ed96d3542eb
```

```bash
# List Docker networks
docker network ls
```

```
NETWORK ID          NAME                DRIVER              SCOPE
78ef21b636ca        bridge              bridge              local
bf8ed6c46a87        host                host                local
bb922a782335        none                null                local
5d13eb305eec        official            macvlan             local
```

```bash
# View our network
docker network inspect official
```

```json
[
    {
        "Name": "official",
        "Id": "5d13eb305eecdd13c9535d666819c3cdc3f7bc84f1a75e9f8cd63ed96d3542eb",
        "Created": "2023-11-29T04:04:41.301166739Z",
        "Scope": "local",
        "Driver": "macvlan",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.10.1.0/24",
                    "IPRange": "10.10.1.3/24"
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
