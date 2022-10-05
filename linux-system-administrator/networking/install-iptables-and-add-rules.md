# Iptables Httpd & Nginx

## Task

> We have one of our websites up and running on our Nautilus infrastructure in Stratos DC. Our security team has raised a concern that right now Apache’s port i.e `5001` is open for all since there is no firewall installed on these hosts. So we have decided to add some security layer for these hosts and after discussions and recommendations we have come up with the following requirements:<br><br>Install `iptables` and all its dependencies on each app host.<br>Block incoming port `3001` on all apps for everyone except for LBR host.<br>Make sure the rules remain, even after system reboot.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `iptables` overview https://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/ & https://www.digitalocean.com/community/tutorials/how-the-iptables-firewall-works
* Check the default policy chain behaviour, standard is to ACCEPT  - https://www.digitalocean.com/community/tutorials/how-to-list-and-delete-iptables-firewall-rules
* Insert at postion 1 means the first rule AFTER the policy rules - https://serverfault.com/questions/163111/allow-traffic-to-from-specific-ip-with-iptables
* Saving `iptables` rules https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Get root on thor
sudo -i

# Check O/S, it was CentOS 7
cat /etc/*release*

# Install bind-utils for resolution like nslookup, host.
yum install -y bind-utils
```

```
...
Installed:
  bind-utils.x86_64 32:9.11.4-26.P2.el7_9.10
...
```

```bash
# Find the IP address of the LBR server
host stlb01
```

```
stlb01 has address 172.17.0.8
```

```bash
# Check thor /etc/hosts for LBR server
grep -i stlb01 /etc/hosts
```

```
172.16.238.14   stlb01
```

The IP addresses are different to lets take a closer look.

```bash
# Connect to stlb01
ssh loki@loki@stlb01

# Check the IP addresses, there were 3 interfaces
ip -c -h a
```

```
...
16: eth0@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:10:ee:0e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.14/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
26: eth1@if27: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:10:ef:05 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.239.5/24 brd 172.16.239.255 scope global eth1
       valid_lft forever preferred_lft forever
42: eth2@if43: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:08 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.8/16 brd 172.17.255.255 scope global eth2
       valid_lft forever preferred_lft forever
```

There are 3 IP addresses:
1. 172.16.238.14/24 from `/etc/hosts` on thor
2. 172.16.239.5/24 we haven't seen.
3. 172.17.0.8/16 from `host` output on thor.

```bash
# Connect to stapp01
ssh tony@stapp01

# Get root
sudo -i

# Check O/S, it was CentOS 7
cat /etc/*release*

# Check if firewalld is running, and it wasn't.
systemctl status firewalld
```

```
Unit firewalld.service could not be found.
```

```bash
# Check if iptables is running, it wasn't installed
systemctl status iptables
```

```
Unit iptables.service could not be found.
```

```bash
# Install iptables
yum install -y iptables-services
```

```
...
Installed:
  iptables-services.x86_64 0:1.4.21-35.el7
...
```

```bash
# Automatically start iptables at boot time and start it right now
systemctl enable --now iptables
```

```
Created symlink from /etc/systemd/system/basic.target.wants/iptables.service to /usr/lib/systemd/system/iptables.service.
```

```bash
# Check what is listening, httpd was on 3001
ss -lntp
```

```
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port
...
LISTEN     0      511           *:3001                      *:*                   users:(("httpd",pid=464,fd=3),("httpd",pid=463,fd=3),("httpd",pid=462,fd=3),("httpd",pid=461,fd=3),("httpd",pid=460,fd=3),("httpd",pid=458,fd=3))
...
```

```bash
# View specificaiton list
iptables -S
```

```
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```

```bash
# View table list with numeric ports
iptables -L -n
```

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

```bash
# Allow incoming TCP connections from all 3 LBR server IP address
# Insert at postion 1 means the first rule AFTER the policy rules.
iptables --insert INPUT 1 --protocol tcp --source 172.16.238.14 --dport 3001 --jump ACCEPT
iptables --insert INPUT 2 --protocol tcp --source 172.16.239.5 --dport 3001 --jump ACCEPT
iptables --insert INPUT 3 --protocol tcp --source 172.17.0.8 --dport 3001 --jump ACCEPT

# Reject incoming TCP connections from 0.0.0.0:3001 (everywhere else)
# DROP does it silently, REJECT lets them know the connection was refused.
iptables -I INPUT 4 -p tcp --dport 3001 -j REJECT

# Check our new rules
iptables -L -n
```

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  172.16.238.14        0.0.0.0/0            tcp dpt:3001
ACCEPT     tcp  --  172.16.239.5         0.0.0.0/0            tcp dpt:3001
ACCEPT     tcp  --  172.17.0.8           0.0.0.0/0            tcp dpt:3001
REJECT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:3001 reject-with icmp-port-unreachable
...
```

We can see our accept rules at the top and our deny rule below them.

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
service iptables reload
```

```
Redirecting to /bin/systemctl reload iptables.service
```

```bash
# Check that connections are blocked from thor jumpbox
curl stapp01:3001
curl stapp02:3001
curl stapp03:3001
```

```
curl: (7) Failed connect to stapp01:3001; Connection refused
```

```bash
# Check that connections are allowed from LBR server
ssh loki@stlb01
curl stapp01:3001
curl stapp02:3001
curl stapp03:3001
```

```
<?php
/**
 * Front to the WordPress application. This file doesn't do anything, but loads
 * wp-blog-header.php which does and tells WordPress to load the theme.
 *
 * @package WordPress
 */

/**
 * Tells WordPress to load the WordPress theme and output it.
 *
 * @var bool
 */
define( 'WP_USE_THEMES', true );

/** Loads the WordPress Environment and Template */
require( dirname( __FILE__ ) . '/wp-blog-header.php' );
```

```bash
# Repeat the iptables steps for the other servers and check again.
ssh steve@stapp02
ssh banner@stapp03
```
