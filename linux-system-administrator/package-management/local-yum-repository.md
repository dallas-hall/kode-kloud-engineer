# Local Yum Repository

## Task

> The Nautilus production support team and security team had a meeting last month in which they decided to use local yum repositories for maintaing packages needed for their servers. For now they have decided to configure a local yum repo on Nautilus Backup Server. This is one of the pending items from last month, so please configure a local yum repository on Nautilus Backup Server as per details given below.
>
> a. We have some packages already present at location `/packages/downloaded_rpms/` on Nautilus Backup Server.
>
> b. Create a yum repo named `epel_local` and make sure to set Repository ID to `epel_local`. Configure it to use package's location `/packages/downloaded_rpms/`.
>
> c. Install package `vim-enhanced` from this newly created repo.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How to configure local yum repo.
  * https://www.thegeekdiary.com/how-to-create-yum-repository-in-centos-rhel/

## Steps

```bash
# Connect to backup server.
ssh clint@stbkp01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Create the local repo file
cat > /etc/yum.repos.d/epel_local.repo
```

```
[epel_local]
name=epel_local
baseurl=file:///packages/downloaded_rpms/
enabled=1
gpgcheck=0
```

Close the file with conttrol + d i.e. `^D`.

```bash
# Install wget
dnf install -y vim-enhanced
```

```
...
Installed:
  gpm-libs-1.20.7-17.el8.x86_64
  vim-common-2:8.0.1763-19.el8.4.x86_64
  vim-enhanced-2:8.0.1763-19.el8.4.x86_64
  vim-filesystem-2:8.0.1763-19.el8.4.noarch

Complete!
```

We are done.
