# Create Message Of The Day (MOTD) Banner

## Task

> During the monthly compliance meeting, it was pointed out that several servers in the Stratos DC do not have a valid banner. The security team has provided serveral approved templates which should be applied to the servers to maintain compliance. These will be displayed to the user upon a successful login.<br><br>Update the message of the day on all application and db servers for Nautilus. Make use of the approved template located at `/home/thor/nautilus_banner` on jump host.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How to configure MOTD in CentOS 7 - https://wiki.centos.org/TipsAndTricks/BannerFiles
* Run sudo command via ssh - https://stackoverflow.com/a/10312700

## Steps

```bash
# Create .ssh folder and keys
mkdir -p ~/.ssh
ssh-keygen -t ed25519
```

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/thor/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/thor/.ssh/id_ed25519.
Your public key has been saved in /home/thor/.ssh/id_ed25519.pub.
The key fingerprint is:
SHA256:18HXZAnXRpqTDbImwjdDsjP5gjv4iRBuBuKuEPaBbkg thor@jump_host.stratos.xfusioncorp.com
The key's randomart image is:
+--[ED25519 256]--+
|        . . ..oo*|
|       . = . o.@o|
|        B = = *.o|
|  .    . * * o . |
|+E..  . S o .    |
|B+.... . o       |
|o+=.. o          |
|+o . o o         |
|o.  . o          |
+----[SHA256]-----+
```

```bash
# Copy keys
ssh-copy-id -i ~/.ssh/id_ed25519 tony@stapp01
ssh-copy-id -i ~/.ssh/id_ed25519 steve@stapp02
ssh-copy-id -i ~/.ssh/id_ed25519 banner@stapp03
ssh-copy-id -i ~/.ssh/id_ed25519 peter@stdb01
```

```
...
Number of key(s) added: 1
...
```

```bash
# Copy the files over all app servers
scp ~/nautilus_banner tony@stapp01:~
scp ~/nautilus_banner steve@stapp02:~
scp ~/nautilus_banner banner@stapp03:~
```

```
nautilus_banner                                 100% 2530     7.3MB/s   00:00
```

```bash
# Copy the files over to db server which failed as DB server didn't have scp
scp ~/nautilus_banner peter@stdb01:~
```

```
bash: scp: command not found
lost connection
```

```bash
# Connect to DB server
ssh peter@stdb01

# Find which package install scp
yum provides scp
```

```
...
openssh-clients-7.4p1-21.el7.x86_64 : An open source SSH client applications
Repo        : base
Matched from:
Filename    : /usr/bin/scp

openssh-clients-7.4p1-22.el7_9.x86_64 : An open source SSH client applications
Repo        : updates
Matched from:
Filename    : /usr/bin/scp
```

```bash
# Install scp into DB server
yum install -y openssh-clients
```

```
...
Installed:
  openssh-clients.x86_64 0:7.4p1-22.el7_9

Dependency Installed:
  libedit.x86_64 0:3.0-12.20121213cvs.el7

Dependency Updated:
  openssh.x86_64 0:7.4p1-22.el7_9      openssh-server.x86_64 0:7.4p1-22.el7_9
```

```bash
# Log off sudo and DB server
exit
exit

# Copy the files over to db server
scp ~/nautilus_banner peter@stdb01:~
```

```
nautilus_banner                                 100% 2530     7.3MB/s   00:00
```

```bash
# Overrite /etc/motd requiring sudo password
ssh -t tony@stapp01 'sudo mv /home/tony/nautilus_banner /etc/motd'
ssh -t steve@stapp02 'sudo mv /home/steve/nautilus_banner /etc/motd'
ssh -t banner@stapp03 'sudo mv /home/banner/nautilus_banner /etc/motd'
ssh -t peter@stdb01 'sudo mv /home/peter/nautilus_banner /etc/motd'

# Check banner
ssh tony@stapp01 'cat /etc/motd'
ssh steve@stapp02 'cat /etc/motd'
ssh banner@stapp03 'cat /etc/motd'
ssh peter@stdb01 'cat /etc/motd'
```

```
################################################################################################
  .__   __.      ___      __    __  .___________. __   __       __    __       _______.        #
       |  \ |  |     /   \    |  |  |  | |           ||  | |  |     |  |  |  |     /       |   #
       |   \|  |    /  ^  \   |  |  |  | `---|  |----`|  | |  |     |  |  |  |    |   (----`   #
       |  . `  |   /  /_\  \  |  |  |  |     |  |     |  | |  |     |  |  |  |     \   \       #
       |  |\   |  /  _____  \ |  `--'  |     |  |     |  | |  `----.|  `--'  | .----)   |      #
       |__| \__| /__/     \__\ \______/      |__|     |__| |_______| \______/  |_______/       #
                                                                                               #
                                                                                               #
                                                                                               #
                                                                                               #
                                 # #  ( )                                                      #
                                  ___#_#___|__                                                 #
                              _  |____________|  _                                             #
                       _=====| | |            | | |==== _                                      #
                 =====| |.---------------------------. | |====                                 #
   <--------------------'   .  .  .  .  .  .  .  .   '--------------/                          #
     \                                                             /                           #
      \_______________________________________________WWS_________/                            #
  wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww                        #
wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww                       #
   wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww                         #
                                                                                               #
                                                                                               #
################################################################################################
Warning! All Nautilus systems are monitored and audited. Logoff immediately if you are not authorized!
```
