# Install Git

## Task

> The Nautilus development team shared with the DevOps team requirements for new application development, setting up a Git repository for that project. Create a Git repository on Storage server in Stratos DC as per details given below:<br><br>Install git package using `yum` on Storage server.<br>After that create/init a git repository `/opt/cluster.git` (use the exact name as asked and make sure not to create a bare repository).

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

None

## Steps

```bash
# Connect to the storage server
ssh natasha@ststor01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*release*

# Switch to root.
sudo -i

# Install git
dnf install -y git
```

```
...
Installed:
  git-2.39.3-1.el8_8.x86_64
...
```

```
# Create the repo
git init --bare  /opt/cluster.git
```

```
Initialized empty Git repository in /opt/cluster.git/.git/
```

We are done.
