# Iptables Httpd & httpd

## Task

> We have one of our websites up and running on our Nautilus infrastructure in Stratos DC. Our security team has raised a concern that right now Apache’s port i.e `5004` is open for all since there is no firewall installed on these hosts. So we have decided to add some security layer for these hosts and after discussions and recommendations we have come up with the following requirements:
> * Install `iptables` and all its dependencies on each app host.
> * Block incoming port `5004` on all apps for everyone except for LBR host.
> * Make sure the rules remain, even after system reboot.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `iptables` overview.
  * https://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/
  * https://www.digitalocean.com/community/tutorials/how-the-iptables-firewall-works
* Check the default policy chain behaviour, standard is to ACCEPT
  * https://www.digitalocean.com/community/tutorials/how-to-list-and-delete-iptables-firewall-rules
* Insert at postion 1 means the first rule AFTER the policy rules.
  * https://serverfault.com/questions/163111/allow-traffic-to-from-specific-ip-with-iptables
* Saving `iptables` rules.
  * https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Check thor /etc/hosts for LBR server
grep -i stlb01 /etc/hosts
```

```
172.16.238.14   stlb01
```

The IP address matches https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus#infrastructure-details

```bash
ssh tony@stapp01

# Get root
sudo -i

# Check current Linux version, it was CentOS Stream 8.
cat /etc/*rel*

# Check if firewalld is running, and it wasn't.
systemctl status firewalld

# Check if iptables is running, it wasn't installed.
systemctl status iptables

# Install iptables and iproute for networking commands like `ss`.
dnf install -y iptables-services iproute

# Automatically start iptables at boot time and start it right now
systemctl enable --now iptables

# Check what is listening, httpd was on 5004
ss -lntp
```

```
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port
...
LISTEN   0        511              0.0.0.0:5004             0.0.0.0:*       users:(("httpd",pid=982,fd=3),("httpd",pid=981,fd=3),("httpd",pid=980,fd=3),("httpd",pid=966,fd=3))
...
```

```bash
# View specificaiton list
iptables -S
```

```
# Warning: iptables-legacy tables present, use iptables-legacy to see them
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
# Warning: iptables-legacy tables present, use iptables-legacy to see them
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
iptables --insert INPUT 1 --protocol tcp --source 172.16.238.14 --dport 5004 --jump ACCEPT


# Reject incoming TCP connections from 0.0.0.0:5004 (everywhere else)
# DROP does it silently, REJECT lets them know the connection was refused.
iptables -I INPUT 2 -p tcp --dport 5004 -j REJECT

# Check our new rules
iptables -L -n
```

```
# Warning: iptables-legacy tables present, use iptables-legacy to see them
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  172.16.238.14        0.0.0.0/0            tcp dpt:5004
REJECT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:5004 reject-with icmp-port-unreachable
...
```

We can see our accept rule at the top and our deny rule below them.

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

# Repeat the iptables steps for the other servers and check again.
ssh steve@stapp02
ssh banner@stapp03

# Check that connections are blocked from thor jumpbox
curl stapp01:5004
curl stapp02:5004
curl stapp03:5004
```

```
curl: (7) Failed connect to stapp01:5004; Connection refused
curl: (7) Failed connect to stapp02:5004; Connection refused
curl: (7) Failed connect to stapp03:5004; Connection refused
```

```bash
# Check that connections are allowed from LBR server
ssh loki@stlb01
curl stapp01:5004
curl stapp02:5004
curl stapp03:5004
```
 
They all spew out pages of HTML. We are done.
