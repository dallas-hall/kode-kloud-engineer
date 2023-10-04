# Troubleshoot HAProxy Loadbalancer

## Task

> xFusionCorp Industries has an application running on Nautlitus infrastructure in Stratos Datacenter. The monitoring tool recognised that there is an issue with the haproxy service on LBR server. That needs to fixed to make the application work properly.
>
> Troubleshoot and fix the issue, and make sure haproxy service is running on Nautilus LBR server. Once fixed, make sure you are able to access the website using StaticApp button on the top bar.

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

# Check the service
systemctl status haproxy -l
```

```
● haproxy.service - HAProxy Load Balancer
   Loaded: loaded (/usr/lib/systemd/system/haproxy.service; disabled; vendor preset: disabled)
   Active: inactive (dead)

Oct 04 08:24:03 stlb01.stratos.xfusioncorp.com systemd[1]: Collecting haproxy.service
```

```bash
# The service is down so try starting it and check again.
systemcl start haproxy
systemctl status haproxy -l
```

```
● haproxy.service - HAProxy Load Balancer
   Loaded: loaded (/usr/lib/systemd/system/haproxy.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Wed 2023-10-04 08:24:36 UTC; 1s ago
  Process: 376 ExecStart=/usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid $OPTIONS (code=exited, status=1/FAILURE)
 Main PID: 376 (code=exited, status=1/FAILURE)

Oct 04 08:24:36 stlb01.stratos.xfusioncorp.com haproxy-systemd-wrapper[376]: [ALERT] 276/082436 (377) : parsing [/etc/haproxy/haproxy.cfg:57] : balance only supports 'roundrobin', 'static-rr', 'leastconn', 'source', 'uri', 'url_param', 'hdr(name)' and 'rdp-cookie(name)' options.
Oct 04 08:24:36 stlb01.stratos.xfusioncorp.com haproxy-systemd-wrapper[376]: [ALERT] 276/082436 (377) : Error(s) found in configuration file : /etc/haproxy/haproxy.cfg
Oct 04 08:24:36 stlb01.stratos.xfusioncorp.com haproxy-systemd-wrapper[376]: [ALERT] 276/082436 (377) : Fatal errors found in configuration.
Oct 04 08:24:36 stlb01.stratos.xfusioncorp.com haproxy-systemd-wrapper[376]: haproxy-systemd-wrapper: exit, haproxy RC=1
Oct 04 08:24:36 stlb01.stratos.xfusioncorp.com systemd[1]: Child 376 belongs to haproxy.service
Oct 04 08:24:36 stlb01.stratos.xfusioncorp.com systemd[1]: haproxy.service: main process exited, code=exited, status=1/FAILURE
Oct 04 08:24:36 stlb01.stratos.xfusioncorp.com systemd[1]: haproxy.service changed running -> failed
Oct 04 08:24:36 stlb01.stratos.xfusioncorp.com systemd[1]: Unit haproxy.service entered failed state.
Oct 04 08:24:36 stlb01.stratos.xfusioncorp.com systemd[1]: haproxy.service failed.
Oct 04 08:24:36 stlb01.stratos.xfusioncorp.com systemd[1]: haproxy.service: cgroup is empty
```

The error message mentions a config file parsing error for the balance keyword at line 57.

```bash
# Create a backup
cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.old

# Check the config
vi /etc/haproxy/haproxy.cfg
```

Change

```
backend static
    balance     round
    server      static 127.0.0.1:4331 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance     round
    server  app1 stapp01:8089 check
    server  app2 stapp02:8089 check
    server  app3 stapp03:8089 check
```

to

```
backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance     roundrobin
    server  app1 stapp01:8089 check
    server  app2 stapp02:8089 check
    server  app3 stapp03:8089 check
```

```bash
# Restart the service and check again.
systemcl start haproxy
systemctl status haproxy -l
```

```
● haproxy.service - HAProxy Load Balancer
   Loaded: loaded (/usr/lib/systemd/system/haproxy.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-10-04 08:27:48 UTC; 2min 2s ago
 Main PID: 396 (haproxy-systemd)
   CGroup: /docker/489088f539c270eb7196cd7f04db344f232d96e830898daf4117a8bf92ea69d2/system.slice/haproxy.service
           ├─396 /usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
           ├─397 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
           └─398 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds

Oct 04 08:27:48 stlb01.stratos.xfusioncorp.com systemd[1]: About to execute: /usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid $OPTIONS
Oct 04 08:27:48 stlb01.stratos.xfusioncorp.com systemd[1]: Forked /usr/sbin/haproxy-systemd-wrapper as 396
Oct 04 08:27:48 stlb01.stratos.xfusioncorp.com systemd[1]: haproxy.service changed failed -> running
Oct 04 08:27:48 stlb01.stratos.xfusioncorp.com systemd[1]: Job haproxy.service/start finished, result=done
Oct 04 08:27:48 stlb01.stratos.xfusioncorp.com systemd[1]: Started HAProxy Load Balancer.
Oct 04 08:27:48 stlb01.stratos.xfusioncorp.com systemd[396]: Executing: /usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
Oct 04 08:27:48 stlb01.stratos.xfusioncorp.com haproxy-systemd-wrapper[396]: haproxy-systemd-wrapper: executing /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
```

The service is back up and running.

```bash
# View the port bindings.
ss -lntp
```

```
State      Recv-Q Send-Q                                   Local Address:Port                                                  Peer Address:Port
LISTEN     0      3000                                                 *:80                                                               *:*                   users:(("haproxy",pid=398,fd=5))
LISTEN     0      4096                                        127.0.0.11:36019                                                            *:*
LISTEN     0      128                                                  *:22                                                               *:*                   users:(("sshd",pid=210,fd=3))
LISTEN     0      128                                               [::]:22                                                            [::]:*                   users:(("sshd",pid=210,fd=4))
```

```bash
# Test the service
curl localhost:80
```

```
Welcome to xFusionCorp Industries!
```

We are done.
