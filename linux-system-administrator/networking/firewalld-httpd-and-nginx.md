# FirewallD, httpd, and Nginx.

## Task

> To secure our Nautilus infrastructure in Stratos Datacenter we have decided to install and configure firewalld on all app servers. We have Apache and Nginx services running on these apps. Nginx is running as a reverse proxy server for Apache. We might have more robust firewall settings in the future, but for now we have decided to go with the given requirements listed below:
>
> a. Allow all incoming connections on Nginx port, i.e `80`.
> b. Block all incoming connections on Apache port, i.e `8080`.
> c. All rules must be permanent.
> d. Zone should be public.
> e. If Apache or Nginx services aren't running already, please make sure to start them.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* firewalld
  * https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7
* firewall-cmd
  * https://fedoramagazine.org/control-the-firewall-at-the-command-line/

```bash
# Connect to stapp01
ssh tony@stapp01

# Get root
sudo -i

# Check current Linux version, it was CentOS 7.6
cat /etc/*rel*

# Check services
systemctl status httpd # OK
systemctl status nginx # OK
systemctl status firewalld # Not installed

# Install firewalld
dnf install firewalld -y

# Enable and start the firewall.
systemctl enable --now firewalld

# Install networking tools, e.g. ss
dnf install -y iproute

# View port bindings.
ss -lntp
```

```
State              Recv-Q             Send-Q                         Local Address:Port                           Peer Address:Port             Process
...
LISTEN             0                  511                                  0.0.0.0:80                                  0.0.0.0:*                 users:(("nginx",pid=1002,fd=6)
...
LISTEN             0                  511                                  0.0.0.0:8080                                0.0.0.0:*                 users:(("httpd",pid=701,fd=3),
...
```

We can see nginx listening on 80 and httpd on 8080.

```bash
# Check firewall zones
firewall-cmd --get-default-zone
```

```
public
```

```bash
# Check firewall zones
firewall-cmd --get-active-zones # No active zones.
firewall-cmd --get-zones
```

```
block dmz drop external home internal public trusted work
```

```bash
# Set default zone to public for the network interface
firewall-cmd --permanent --change-interface=eth0 --zone=public

# Restart the firewall.
systemctl restart firewalld
firewall-cmd --get-active-zones
```

```
public
  interfaces: eth0
```

```bash
# Add a new port/protocol permanently
firewall-cmd --permanent --zone=public --add-port=80/tcp
```

```
Success.
```

```bash
systemctl restart firewalld
firewall-cmd --list-ports
```

```
80/tcp.
```

```bash
# Repeat for the rest.
ssh steve@stapp02
ssh banner@stapp03

# Check connection internally on all 3 hosts.
curl localhost:80 # OK for nginx
curl localhost:8080 # OK for httpd.

# Check connection externally - exit out to thor.
curl stapp01:80 # OK for nginx
curl stapp02:80 # OK for nginx
curl stapp03:80 # OK for nginx

curl stapp01:8080 # Times out for httpd.
curl stapp02:8080 # Times out for httpd.
curl stapp03:8080 # Times out for httpd.
```

We are done.
