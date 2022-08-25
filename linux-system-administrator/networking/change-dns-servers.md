# Change DNS Nameserver

## Task

> The system admins team of xFusionCorp Industries has noticed intermittent issues with DNS resolution in several apps . App Server 1 in Stratos Datacenter is having some DNS resolution issues, so we want to add some additional DNS nameservers on this server.<br><br>As a temporary fix we have decided to go with Google public DNS (ipv4). Please make appropriate changes on this server.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Change DNS name server - https://www.cyberciti.biz/faq/change-dns-ip-address-rhel-redhat-linux/
* Google DNS servers - https://developers.google.com/speed/public-dns
* Install `nslookup` and `host` - https://unix.stackexchange.com/questions/164210/nslookup-command-not-found-error-on-rhel-centos-7

## Steps


```bash
# Connect to stapp01
ssh steve@stapp02

# Get root
sudo -i

# Check O/S, it was CentOS 7
cat /etc/*release*

# See current DNS
cat /etc/resolv.conf

# Try to resolve google.com
nslookup google.com
```

```
bash: nslookup: command not found
```

```bash
# Install nslookup, failing due to DNS issues
yum install bind-utils -y
```

```
...
http://mirror.netdepot.com/centos/7.9.2009/os/x86_64/repodata/41232548001a78473ae0f2d4b92e1ec28f7a0593e0495056515887fe2a39b416-filelists.sqlite.bz2: [Errno 14] HTTP Error 404 - Not Found
Trying other mirror.
...
```

DNS is broken so we can't install any thing from the internet.

```bash
# Check current DNS file
cat /etc/resolv.conf
```

```
search stratos.xfusioncorp.com
nameserver 127.0.0.11
options ndots:0
```

```bash
# Create a backup
cp /etc/resolv.conf /etc/resolv.conf.bak

# Change DNS
vi /etc/resolv.conf
```

```
search stratos.xfusioncorp.com
#nameserver 127.0.0.11
nameserver 8.8.8.8
nameserver 8.8.4.4
options ndots:0
```

```bash
# Install nslookup, works after fixing DNS
yum install bind-utils -y
```

```
...
Installed:
  bind-utils.x86_64 32:9.11.4-26.P2.el7_9.9

Dependency Installed:
  GeoIP.x86_64 0:1.5.0-14.el7
  bind-libs.x86_64 32:9.11.4-26.P2.el7_9.9
  bind-libs-lite.x86_64 32:9.11.4-26.P2.el7_9.9
  geoipupdate.x86_64 0:2.5.0-1.el7
...
```


```bash
# Try to resolve google.com
nslookup google.com
```

```
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   google.com
Address: 74.125.70.138
Name:   google.com
Address: 74.125.70.102
Name:   google.com
Address: 74.125.70.101
Name:   google.com
Address: 74.125.70.100
Name:   google.com
Address: 74.125.70.139
Name:   google.com
Address: 74.125.70.113
Name:   google.com
Address: 2607:f8b0:4001:c1c::8a
Name:   google.com
Address: 2607:f8b0:4001:c1c::64
Name:   google.com
Address: 2607:f8b0:4001:c1c::8b
Name:   google.com
Address: 2607:f8b0:4001:c1c::66
```

This is working after fixing DNS.

```bash
# Try to resolve google.com
host google.com
```

```
google.com has address 172.217.212.102
google.com has address 172.217.212.100
google.com has address 172.217.212.139
google.com has address 172.217.212.113
google.com has address 172.217.212.138
google.com has address 172.217.212.101
google.com has IPv6 address 2607:f8b0:4001:c5a::8a
google.com has IPv6 address 2607:f8b0:4001:c5a::71
google.com has IPv6 address 2607:f8b0:4001:c5a::65
google.com has IPv6 address 2607:f8b0:4001:c5a::8b
google.com mail is handled by 10 smtp.google.com.
```

This is working after fixing DNS.


```bash
# Try to ping google.com
ping -c 3 google.com
```

```
PING google.com (172.217.212.100) 56(84) bytes of data.
64 bytes from jq-in-f100.1e100.net (172.217.212.100): icmp_seq=1 ttl=113 time=0.600 ms
64 bytes from jq-in-f100.1e100.net (172.217.212.100): icmp_seq=2 ttl=113 time=0.408 ms
64 bytes from jq-in-f100.1e100.net (172.217.212.100): icmp_seq=3 ttl=113 time=0.435 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2009ms
rtt min/avg/max/mdev = 0.408/0.481/0.600/0.084 ms
```

This is working after fixing DNS.