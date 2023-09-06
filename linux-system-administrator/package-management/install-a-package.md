# Install A Package

## Task

> As per new application requirements shared by the Nautilus project development team, serveral new packages need to be installed on all app servers in Stratos Datacenter. Most of them are completed except for `git`.<br><br>Therefore, install the `git` package on all app-servers.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None.

## Steps

Follow [Passwordless ssh setup](networking/passwordless-ssh-access.md) for every user.

```bash
# Connect to the first app server
ssh tony@stapp01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Install package
dnf install -y git
```

```
...
Installed:
  emacs-filesystem-1:26.1-11.el8.noarch                                         git-2.39.3-1.el8_8.x86_64
  git-core-2.39.3-1.el8_8.x86_64                                                git-core-doc-2.39.3-1.el8_8.noarch
...
```

```bash
# Repeat for the other app servers.
ssh steve@stapp02
ssh banner@stapp03
```

We are done.
