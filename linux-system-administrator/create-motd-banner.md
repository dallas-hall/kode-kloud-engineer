# Create Message Of The Day (MOTD) Banner

## Task

> During the monthly compliance meeting, it was pointed out that several servers in the Stratos DC do not have a valid banner. The security team has provided serveral approved templates which should be applied to the servers to maintain compliance. These will be displayed to the user upon a successful login.<br><br>Update the message of the day on all application and db servers for Nautilus. Make use of the approved template located at `/home/thor/nautilus_banner` on jump host.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How to configure MOTD in CentOS 7
  * https://wiki.centos.org/TipsAndTricks/BannerFiles
* Run sudo command via ssh
  * https://stackoverflow.com/a/10312700

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Copy the files over all app servers
scp ~/nautilus_banner tony@stapp01:~
scp ~/nautilus_banner steve@stapp02:~
scp ~/nautilus_banner banner@stapp03:~
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

We are done.
