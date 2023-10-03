# Install & Configure HAProxy Loadbalancer

## Task

> There is a static website running in Stratos Datacenter. They have already configured the app servers and code is already deployed there. To make it work properly, they need to configure LBR server. There are number of options for that, but team has decided to go with HAproxy. FYI, apache is running on port `8082` on all app servers. Complete this task as per below details.
>
> a. Install and configure HAproxy on LBR server using `yum` only and make sure all app servers are added to HAproxy load balancer. HAproxy must serve on default http port (Note: Please do not remove stats socket `/var/lib/haproxy/stats` entry from haproxy default config.).
> b. Once done, you can access the website using StaticApp button on the top bar.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* HAProxy port.
  * https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers#set-the-listening-ip-address-and-port
* HAProxy backend.
  * https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers#define-a-backend

## Steps

```bash
# Connect to the load balancer server.
ssh loki@stlb01

# Get root
sudo -i

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Install HAProxy.
dnf install -y haproxy

# Enable and start HAProxy
systemctl enable --now haproxy

# Instal IP Route for ss
dnf install -y iproute

# Check port bindings.
ss -lntp
```

```
State          Recv-Q         Send-Q                 Local Address:Port                    Peer Address:Port         Process
LISTEN         0              3000                         0.0.0.0:5000                         0.0.0.0:*             users:(("haproxy",pid=1120,fd=5))
LISTEN         0              128                          0.0.0.0:22                           0.0.0.0:*             users:(("sshd",pid=484,fd=3))
LISTEN         0              4096                      127.0.0.11:42457                        0.0.0.0:*
LISTEN         0              128                             [::]:22                              [::]:*             users:(("sshd",pid=484,fd=4))
```

HAProxy is listening on 5000.

```bash
# Configure HAProxy with app servers.
vi /etc/haproxy/haproxy.cfg
```

Change the line `bind *:5000` to `bind *:80`.

Change the lines

```
    server  app1 127.0.0.1:5004 check
    server  app2 127.0.0.1:5004 check
    server  app3 127.0.0.1:5004 check
    server  app4 127.0.0.1:5004 check
```

to

```
    server  stapp01 172.16.238.10:8082 check
    server  stapp02 172.16.238.11:8082 check
    server  stapp03 172.16.238.12:8082 check
```

Press `:x` to save and quit.

```bash
# Restart HAProxy
systemctl restart haproxy

# Test the connection
curl -v localhost:80
```

```
* Rebuilt URL to: localhost:80/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: localhost
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Tue, 03 Oct 2023 08:32:48 GMT
< Server: Apache/2.4.37 (centos)
< Last-Modified: Tue, 03 Oct 2023 08:11:58 GMT
< ETag: "23-606cb71d3e954"
< Accept-Ranges: bytes
< Content-Length: 35
< Content-Type: text/html; charset=UTF-8
<
Welcome to xFusionCorp Industries!
* Connection #0 to host localhost left intact
```

We are done.
