# Troubleshooting Failed httpd Process

## Task

> The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems. One of the monitoring systems reported about Apache service unavailability on one of the app servers in Stratos DC.
>
> Identify the faulty app host and fix the issue. Make sure Apache service is up and running on all app hosts. They might not hosted any code yet on these servers so you need not to worry about if Apache isn't serving any pages or not, just make sure service is up and running. Also, make sure Apache is running on port `6000` on all app servers.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None.

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Check app servers
ssh tony@stapp01 curl -kLv localhost:6000
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 6000 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 6000 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:6000
> Accept: */*
> 
{ [data not shown]
1220 stapp01.stratos.xfusioncorp.com ESMTP Sendmail 8.14.7/8.14.7; Mon, 8 Jan 2024 05:17:38 GMT
421 4.7.0 stapp01.stratos.xfusioncorp.com Rejecting open proxy localhost [127.0.0.1]
00   182    0   182    0     0  27978      0 --:--:-- --:--:-- --:--:-- 30333
* Connection #0 to host localhost left intact
```

</details>


Looks like we have a problem.

```bash
# Connect to the server and investigate
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
cat /etc/*rel*

# Get root
sudo -i

# Check httpd service, it was dead.
systemctl status httpd

# Start the server
systemctl enable --now httpd
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
Job for httpd.service failed because the control process exited with error code. See "systemctl status httpd.service" and "journalctl -xe" for details.
```

There is still an error.

```bash
# View the logs
journalctl -xe -u httpd
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
-- Logs begin at Mon 2024-01-08 05:15:30 UTC, end at Mon 2024-01-08 05:19:2
0 UTC. --
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com systemd[1]: Starting The Ap
ache HTTP Server...
-- Subject: Unit httpd.service has begun start-up
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit httpd.service has begun starting up.
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com httpd[910]: AH00558: httpd:
 Could not reliably determine the server's fully qualified domain name, usi
ng stapp01.stratos.xfusioncorp.com. Set the 'ServerName' directive globally
 to suppress this message
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com httpd[910]: (98)Address alr
eady in use: AH00072: make_sock: could not bind to address 0.0.0.0:6000
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com httpd[910]: no listening so
ckets available, shutting down
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com httpd[910]: AH00015: Unable
 to open logs
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.ser
vice: main process exited, code=exited, status=1/FAILURE
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com kill[911]: kill: cannot fin
d process ""
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.ser
vice: control process exited, code=exited status=1
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com systemd[1]: Failed to
 start The Apache HTTP Server.
-- Subject: Unit httpd.service has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit httpd.service has failed.
-- 
-- The result is failed.
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com systemd[1]: Unit http
d.service entered failed state.
Jan 08 05:19:20 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.ser
vice failed.
```

</details>

There is another service listening on that port.

```bash
# View what is running on 6000
ss -lntp | grep 6000
```

```
LISTEN     0      10     127.0.0.1:6000                     *:*                   users:(("sendmail",pid=593,fd=4))
```

```bash
# Stop the service listening on 6000
systemctl disable --now sendmail
```

```
Removed symlink /etc/systemd/system/multi-user.target.wants/sm-client.service.
Removed symlink /etc/systemd/system/multi-user.target.wants/sendmail.service.
```

Could also update the `sendmail` port via `/etc/mail/sendmail.mc` and the default port is 25.

```bash
# Start httpd
systemctl start httpd

# Testing locally worked.
curl -kLv localhost:6000

# Testing externally worked.
exit
exit
curl -kLv stapp01:6000

# Check next server, it worked fine.

curl -kLv stapp02:6000
# Check next server, it worked fine.
curl -kLv stapp03:6000
```

Everything is working now, we are done.
