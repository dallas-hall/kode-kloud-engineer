# Log Rotate

## Task

> The Nautilus DevOps team is ready to launch a new application, which they will deploy on app servers in Stratos Datacenter. They are expecting significant traffic/usage of `squid` on app servers after that. This will generate massive logs, creating huge log files. To utilise the storage efficiently, they need to compress the log files and need to rotate old logs. Check the requirements shared below:<br><br>a. In all app servers install `squid` package.<br>b. Using `logrotate` configure `squid` logs rotation to monthly and keep only 3 rotated logs. If by default log rotation is set, then please update configuration as needed.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Logrotate - https://www.redhat.com/sysadmin/setting-logrotate & https://www.digitalocean.com/community/tutorials/how-to-manage-logfiles-with-logrotate-on-ubuntu-16-04
* `man logrotate`

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Connect to the first app server.
ssh tony@stapp01

# Switch to root.
sudo -i

# Install the app.
yum install -y squid
```

```
...
Installed:
  squid.x86_64 7:3.5.20-17.el7_9.7
...
```

```bash
# Start and enable squid.
systemctl enable --now squid
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/squid.service to /usr/lib/systemd/system/squid.service.
```

```bash
# Take a backup of the pre-existing squid logrotate file.
cp /etc/logrotate.d/squid /etc/logrotate.d/squid.bak

# Inspect the file to figure out changes.
cat /etc/logrotate.d/squid
```

```
/var/log/squid/*.log {
    weekly
    rotate 5
    compress
    notifempty
    missingok
    nocreate
    sharedscripts
    postrotate
      # Asks squid to reopen its logs. (logfile_rotate 0 is set in squid.conf)
      # errors redirected to make it silent if squid is not running
      /usr/sbin/squid -k rotate 2>/dev/null
      # Wait a little to allow Squid to catch up before the logs is compressed
      sleep 1
    endscript
}
```

```bash
# Configure the file to the requirements.
sed -i -r 's/weekly/monthly/g' /etc/logrotate.d/squid
sed -i -r 's/rotate 5/rotate 3/g' /etc/logrotate.d/squid

# Inspect the changes
cat /etc/logrotate.d/squid
```

```
/var/log/squid/*.log {
    monthly
    rotate 3
    compress
    notifempty
    missingok
    nocreate
    sharedscripts
    postrotate
      # Asks squid to reopen its logs. (logfile_rotate 0 is set in squid.conf)
      # errors redirected to make it silent if squid is not running
      /usr/sbin/squid -k rotate 2>/dev/null
      # Wait a little to allow Squid to catch up before the logs is compressed
      sleep 1
    endscript
}
```

```bash
# Repeat for the rest of the servers
ssh steve@stapp02
ssh banner@stapp03
```
