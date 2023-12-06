# Network Troubleshooting

## Task

> Our monitoring tool has reported an issue in Stratos Datacenter. One of our app servers has an issue, as its Apache service is not reachable on port `8085` (which is the Apache port). The service itself could be down, the firewall could be at fault, or something else could be causing the issue.
>
> Use tools like `telnet`, `netstat`, etc. to find and fix the issue. Also make sure Apache is reachable from the jump host without compromising any security settings.
>
> Once fixed, you can test the same using command `curl http://stapp01:8085` command from jump host.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `iptables` overview
  * https://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/
  * https://www.digitalocean.com/community/tutorials/how-the-iptables-firewall-works
* Saving `iptables` rules
  * https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands

## Steps


Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Test connectivity from the jump box.
```

```
* Rebuilt URL to: http://stapp01:8085/
*   Trying 172.16.238.10...
* TCP_NODELAY set
* connect to 172.16.238.10 port 8085 failed: No route to host
* Failed to connect to stapp01 port 8085: No route to host
* Closing connection 0
curl: (7) Failed to connect to stapp01 port 8085: No route to host
```

```bash
# Connect to the first app server
ssh tony@stapp01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Install network troubleshooting tools
dnf install -y iproute net-tools

# Check the running services and their port bindings for listening TCP.
ss -lntp
```

```
State          Recv-Q         Send-Q                 Local Address:Port                    Peer Address:Port         Process                                    
LISTEN         0              10                         127.0.0.1:8085                         0.0.0.0:*             users:(("sendmail",pid=708,fd=4))         
LISTEN         0              128                          0.0.0.0:22                           0.0.0.0:*             users:(("sshd",pid=554,fd=3))             
LISTEN         0              4096                      127.0.0.11:34711                        0.0.0.0:*                                                       
LISTEN         0              128                             [::]:22                              [::]:*             users:(("sshd",pid=554,fd=4)) 
```

We can see that there is no `httpd` process listening on TCP 8085 but we can see the `sendmail` process listening on that port.

```bash
# Check the service.
systemctl status httpd
```

```
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Wed 2023-12-06 04:10:49 UTC; 6min ago
     Docs: man:httpd.service(8)
  Process: 771 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
 Main PID: 771 (code=exited, status=1/FAILURE)
   Status: "Reading configuration..."

Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com httpd[771]: (98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:8085
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com httpd[771]: no listening sockets available, shutting down
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com httpd[771]: AH00015: Unable to open logs
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: Child 771 belongs to httpd.service.
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: Failed with result 'exit-code'.
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: Changed start -> failed
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: Job httpd.service/start finished, result=failed
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com systemd[1]: Failed to start The Apache HTTP Server.
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: Unit entered failed state.
```

We can see the `httpd` is broken.

```bash
# Check the httpd logs
journalctl -u httpd
```

```
...
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com systemd[771]: httpd.service: Executing: /usr/sbin/httpd -DFOREGROUND
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 771 (RELOADING=1, STATUS=Re
ading configuration...)
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com httpd[771]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using stapp01
.stratos.xfusioncorp.com. Set the 'ServerName' directive globally to suppress this message
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com httpd[771]: (98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:8085
Dec 06 04:10:49 stapp01.stratos.xfusioncorp.com httpd[771]: no listening sockets available, shutting down
...
```

The logs confirm that `sendmail` is blocking `httpd`.

```bash
# Shut down sendmail and stop it from starting.
systemctl stop sendmail
systemctl disable sendmail

# Restart httpd
systemctl restart httpd

# Check the service, it was running.
systemctl status httpd

# Checking connectivity from stapp01 worked, but the jump box failed.
curl http://stapp01:8085

# Check firewall
systemctl status firewalld # Doesn't exist
systemctl status iptables # Running

# Check firewall rules, there is no rule allowing httpd traffic on port 8085.
iptables -L -n

# Turn off firewall and test from thor
systemctl stop iptables
curl http://stapp01:8085 # Works, firewall problem

# Add a specific rule for httpd
systemctl restart iptables
iptables --insert INPUT 1 --protocol tcp --dport 8085 --jump ACCEPT
service iptables save
systemctl reload iptables

# Test changes from thor was working.
curl http://stapp01:8085
```

We are done.
