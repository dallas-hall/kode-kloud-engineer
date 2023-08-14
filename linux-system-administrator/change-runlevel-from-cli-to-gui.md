# Changing Runlevel From CLI To GUI

## Task

> New tools have been installed on the app server in Stratos Datacenter. Some of these tools can only be managed from the graphical user interface. Therefore, there are requirements for these app servers.<br><br>On all App servers in Stratos Datacenter change the default runlevel so that they can boot in GUI (graphical user interface) by default. Please do not try to reboot these servers

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Changing runlevels - https://www.thegeekdiary.com/centos-rhel-7-how-to-change-runlevels-in-rhel7-with-systemd/

## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS Steam 8.
cat /etc/*rel*

# Switch to root
sudo -i

# See current
systemctl get-default
```

```
multi-user.target
```

The above output means that a multi-user text only CLI is being used.

```bash
# Show all runlevels
systemctl list-units --type=target --all
```

```
  UNIT                   LOAD      ACTIVE   SUB    DESCRIPTION
  basic.target           loaded    active   active Basic System
● cryptsetup.target      not-found inactive dead   cryptsetup.target
  emergency.target       loaded    inactive dead   Emergency Mode
  getty-pre.target       loaded    inactive dead   Login Prompts (Pre)
● getty.target           masked    inactive dead   getty.target
  graphical.target       loaded    inactive dead   Graphical Interface
  local-fs-pre.target    loaded    inactive dead   Local File Systems (Pre)
  local-fs.target        loaded    active   active Local File Systems
  multi-user.target      loaded    active   active Multi-User System
  network-online.target  loaded    active   active Network is Online
  network-pre.target     loaded    inactive dead   Network (Pre)
  network.target         loaded    inactive dead   Network
  nss-user-lookup.target loaded    inactive dead   User and Group Name Lookups
  paths.target           loaded    active   active Paths
  remote-fs-pre.target   loaded    inactive dead   Remote File Systems (Pre)
  remote-fs.target       loaded    active   active Remote File Systems
  rescue.target          loaded    inactive dead   Rescue Mode
  shutdown.target        loaded    inactive dead   Shutdown
  slices.target          loaded    active   active Slices
  sockets.target         loaded    active   active Sockets
  sshd-keygen.target     loaded    active   active sshd-keygen.target
  swap.target            loaded    active   active Swap
  sysinit.target         loaded    active   active System Initialization
  time-sync.target       loaded    inactive dead   System Time Synchronized
  timers.target          loaded    active   active Timers
  umount.target          loaded    inactive dead   Unmount All Filesystems

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

26 loaded units listed.
To show all installed unit files use 'systemctl list-unit-files'.
```

```bash
# Change to GUI
systemctl set-default graphical.target
```

```
Removed /etc/systemd/system/default.target.
Created symlink /etc/systemd/system/default.target → /usr/lib/systemd/system/graphical.target.
```

```bash
# Check it was changed
systemctl get-default
```

```
graphical.target
```

The above output means that a multi-user graphical mode is in use.

```bash
# Repeat for the rest of the servers
ssh steve@stapp02
ssh banner@stapp03
```

We are done.
