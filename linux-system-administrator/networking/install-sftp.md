# Install SFTP

## Task

> Some of the developers from Nautilus project team have asked for SFTP access to at least one of the app server in Stratos DC. After going through the requirements, the system admins team has decided to configure the SFTP server on App Server 3 server in Stratos Datacenter. Please configure it as per the following instructions:
>
> * Create a SFTP user `yousuf` and set its password to `BruCStnMT5`. There is already a group called `ftp`, you can utilise the same.
> * Password authentication should be enabled for this user.
> * SFTP user should only be allowed to make SFTP connections.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* SFTP
  * https://www.ssh.com/academy/ssh/sftp-ssh-file-transfer-protocol
  * https://tecadmin.net/setup-sftp-server-on-centos/
  * https://unix.stackexchange.com/a/542507

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Connect to the first app server
ssh banner@stapp03

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check if sftp is install, it was.
which sftp

# Creae the sftp user.
useradd yousuf
usermod -aG ftp yousuf
passwd yousuf
```

```
Changing password for user yousuf.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

```bash
# Back up sshd config file
cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.old

# Update sshd config file
cat >> /etc/ssh/sshd_config
```

```
Match User yousuf
    ForceCommand internal-sftp
    PasswordAuthentication yes
    ChrootDirectory /home/yousuf
    PermitTunnel no
    AllowAgentForwarding no
    AllowTcpForwarding no
    X11Forwarding no
```

Close the file with control + d i.e. `^D`

```bash
# Change directory permissions for sftp.
chown root: /home/yousuf
chmod 755 /home/yousuf

# Create a scratch space for the sftp user.
mkdir /home/yousuf/tmp
chown yousuf: /home/yousuf/tmp

# Check the sshd config.
systemctl reload sshd

# Restart sshd after the config was validated.
systemctl restart sshd

# Test the connection from the Thor jumpbox.
sftp yousuf@stapp03
```

```
yousuf@stapp03's password: 
Connected to stapp03.
sftp>
```

We are done.
