# Iptables Httpd & Nginx

## Task

> We have a backup management application UI hosted on Nautilus's backup server in Stratos DC. That backup management application code is deployed under Apache on the backup server itself, and Nginx is running as a reverse proxy on the same server. Apache and Nginx ports are `3001` and `8099`, respectively. We have `iptables` firewall installed on this server. Make the appropriate changes to fulfill the requirements mentioned below:<br><br>We want to open all incoming connections to Nginx's port and block all incoming connections to Apache's port. Also make sure rules are permanent.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `iptables` overview https://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/ & https://www.digitalocean.com/community/tutorials/how-the-iptables-firewall-works
* Saving `iptables` rules https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands

## Steps

```bash
# Connect to stapp01
ssh clint@stbkp01

# Get root
sudo -i

# Check O/S, it was CentOS 7
cat /etc/*release*

# Check the apps listening for TCP connections
ss -lntp
```

```
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port
...
LISTEN     0      511           *:3001                      *:*                   users:(("httpd",pid=536,fd=3),("httpd",pid=535,fd=3),("httpd",pid=534,fd=3),("httpd",pid=533,fd=3),("httpd",pid=532,fd=3),("httpd",pid=531,fd=3))
LISTEN     0      511           *:8099                      *:*                   users:(("nginx",pid=589,fd=6),("nginx",pid=588,fd=6),("nginx",pid=587,fd=6),("nginx",pid=586,fd=6),("nginx",pid=585,fd=6),("nginx",pid=584,fd=6),("nginx",pid=583,fd=6),("nginx",pid=582,fd=6),("nginx",pid=581,fd=6),("nginx",pid=580,fd=6),("nginx",pid=579,fd=6),("nginx",pid=578,fd=6),("nginx",pid=577,fd=6),("nginx",pid=576,fd=6),("nginx",pid=575,fd=6),("nginx",pid=574,fd=6),("nginx",pid=573,fd=6),("nginx",pid=572,fd=6),("nginx",pid=571,fd=6),("nginx",pid=570,fd=6),("nginx",pid=569,fd=6),("nginx",pid=568,fd=6),("nginx",pid=567,fd=6),("nginx",pid=566,fd=6),("nginx",pid=565,fd=6),("nginx",pid=564,fd=6),("nginx",pid=563,fd=6),("nginx",pid=562,fd=6),("nginx",pid=561,fd=6),("nginx",pid=560,fd=6),("nginx",pid=559,fd=6),("nginx",pid=558,fd=6),("nginx",pid=557,fd=6),("nginx",pid=556,fd=6),("nginx",pid=555,fd=6),("nginx",pid=554,fd=6),("nginx",pid=553,fd=6))
...
```

```bash
# Check connectivty from thor jumpbox to backup server for httpd
curl -v stbkp01:3001
```

```
curl -v stbkp01:3001
* About to connect() to stbkp01 port 3001 (#0)
*   Trying 172.16.238.16...
* Connected to stbkp01 (172.16.238.16) port 3001 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: stbkp01:3001
> Accept: */*
...
```

Connection is open.


```bash
# Check connectivty from thor jumpbox to backup server for nginx
curl -v stbkp01:8099
```

```
* About to connect() to stbkp01 port 8099 (#0)
*   Trying 172.16.238.16...
* Connected to stbkp01 (172.16.238.16) port 8099 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: stbkp01:8099
> Accept: */*
...
```

Connection is open.

```bash
# Check if iptables is installed and running
systemctl status iptables
```

```
● iptables.service - IPv4 firewall with iptables
   Loaded: loaded (/usr/lib/systemd/system/iptables.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
```

```bash
# Automatically start iptables at boot time and start it right now
systemctl enable iptables
systemctl start iptables

# Accept incoming TCP connections from all IPs for nginx on port 8099
# Insert at postion 1 means the first rule AFTER the policy rules.
iptables --insert INPUT 1 --protocol tcp --dport 8099 --jump ACCEPT

# Reject incoming TCP connections from all IPs for httpd on port 3001
# DROP does it silently, REJECT lets them know the connection was refused.
iptables -I INPUT 2 -p tcp --dport 3001 -j REJECT

# Check our rules
iptables -L -n
```

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8099
REJECT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:3001 reject-with icmp-port-unreachable
...
```

We can see our rules at the top.

```bash
# Save the updated rules
service iptables save
```

```
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
```

```bash
# Reload the service
systemctl reload iptables

# Check connectivty from thor jumpbox to backup server for nginx
curl -v stbkp01:8099
```

```
* About to connect() to stbkp01 port 8099 (#0)
*   Trying 172.16.238.16...
* Connected to stbkp01 (172.16.238.16) port 8099 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: stbkp01:8099
> Accept: */*
>
< HTTP/1.1 403 Forbidden
< Server: nginx/1.16.1
< Date: Thu, 08 Sep 2022 09:28:17 GMT
< Content-Type: text/html
< Content-Length: 153
< Connection: keep-alive
<
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.16.1</center>
</body>
</html>
* Connection #0 to host stbkp01 left intact
```

Requests to the `nginx` reverse proxy are allowed like we wanted

```bash
# Check connectivty from thor jumpbox to backup server for httpd
curl -v stbkp01:3001
```

```
* About to connect() to stbkp01 port 3001 (#0)
*   Trying 172.16.238.16...
* Connection refused
* Failed connect to stbkp01:3001; Connection refused
* Closing connection 0
curl: (7) Failed connect to stbkp01:3001; Connection refused
```

Requests to `httpd` webserver are blocked like we wanted