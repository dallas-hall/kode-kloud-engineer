# Local Yum Repository

## Task

> As per new application requirements shared by the Nautilus project development team, serveral new packages need to be installed on all app servers in Stratos Datacenter. Most of them are completed except for `wget`.<br><br>Therefore, install the `wget` package on all app-servers.

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

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Install package
yum install -y wget
```

```
...
Installed:
  wget.x86_64 0:1.14-18.el7_6.1
...
```

```bash
# Repeat for the other app servers.
ssh steve@stapp02
ssh banner@stapp03
```
