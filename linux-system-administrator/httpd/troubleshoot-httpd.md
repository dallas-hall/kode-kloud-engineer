# Troubleshoot httpd

## Task

> xFusionCorp Industries uses some monitoring tools to check the status of every service, application, etc running on the systems. Recently, the monitoring system identified that Apache service is not running on some of the Nautilus Application Servers in Stratos Datacenter.
>
> 1. Identify the faulty Nautilus Application Server and fix the issue. Also, make sure Apache service is up and running on all Nautilus Application Servers. Do not try to stop any kind of firewall that is already running.
> 2. Apache is running on `3000` port on all Nautilus Application Servers and its document root must be `/var/www/html` on all app servers.
> 3. Finally you can test from jump host using `curl` command to access Apache on all app servers and it should be reachable and you should get some static page. E.g. `curl http://172.16.238.10:3000/`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

None.

## Steps

```bash
# Check the app servers. They all failed with connection refused.
curl stapp01:3000
curl stapp02:3000
curl stapp03:3000
```

```
curl: (7) Failed to connect to stapp01 port 3000: Connection refused
curl: (7) Failed to connect to stapp02 port 3000: Connection refused
curl: (7) Failed to connect to stapp03 port 3000: Connection refused
```

```bash
# Connect to app 1.
ssh tony@stapp01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check the service, it was dead.
systemctl status httpd

# Try starting the service, it didn't start.
systemctl start httpd

# Check the journal logs.
journalctl -u httpd
```

The interesting line is:

```
Oct 11 08:17:57 stapp01.stratos.xfusioncorp.com httpd[1051]: AH00526: Syntax error on line 45 of /etc/httpd/conf/httpd.conf:
```

```bash
# Take a backup
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.old

# Edit the file.
vi /etc/httpd/conf/httpd.conf
```

Replace `"Listen 3000"` with `Listen 3000` and save and close the file `:x`

```bash
# Try starting the service, it didn't start.
systemctl start httpd

# Check the journal logs.
journalctl -u httpd
```

The interesting lines are:

```
Oct 11 08:21:44 stapp01.stratos.xfusioncorp.com httpd[1110]: AH00526: Syntax error on line 122 of /etc/httpd/conf/httpd.conf:
Oct 11 08:21:44 stapp01.stratos.xfusioncorp.com httpd[1110]: DocumentRoot '/var/www/html;' is not a directory, or is not readable
```

Replace `DocumentRoot /var/www/html;` with `DocumentRoot /var/www/html` and save and close the file `:x`

```bash
# Try starting the service, it started.
systemctl start httpd

# Test the local connection. It worked.
curl localhost:3000

# Log out of root.
exit

# Log out of the stapp01 server.
exit

# Test the connection from the jump box. It worked.
curl stapp01:3000
```

```
Welcome to xFusionCorp Industries!
```

Do the same checks for stapp02.

```bash
# Connect to app 2.
ssh steve@stapp02

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check the service, it was dead.
systemctl status httpd

# Try starting the service, it didn't start.
systemctl start httpd

# Check the journal logs.
journalctl -u httpd
```

The interesting line is:

```
Oct 11 08:27:07 stapp02.stratos.xfusioncorp.com httpd[1137]: httpd: Syntax error on line 34 of /etc/httpd/conf/httpd.conf: ServerRoot must be a valid directory
```

```bash
# Take a backup
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.old

# Edit the file.
vi /etc/httpd/conf/httpd.conf
```

Replace `ServerRoot "/etc/httpd;"` with `ServerRoot /etc/httpd` and save and close the file `:x`

```bash
# Try starting the service, it started.
systemctl start httpd

# Test the local connection. It worked.
curl localhost:3000

# Log out of root.
exit

# Log out of the stapp02 server.
exit

# Test the connection from the jump box. It worked.
curl stapp02:3000
```

```
Welcome to xFusionCorp Industries!
```

Do the same checks for stapp03.

```bash
# Connect to app 3.
ssh banner@stapp03

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check the service, it was dead.
systemctl status httpd

# Try starting the service, it started.
systemctl start httpd

# Test the local connection. It worked.
curl localhost:3000

# Log out of root.
exit

# Log out of the stapp01 server.
exit

# Test the connection from the jump box. It worked.
curl stapp03:3000
```

```
Welcome to xFusionCorp Industries!
```

We are done.
