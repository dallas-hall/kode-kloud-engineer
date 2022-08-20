# User Without Home Folder

## Task

> The system admins team of xFusionCorp Industries has set up a new tool on all app servers, as they have a requirement to create a service user account that will be used by that tool. They are finished with all apps except for App Server 1 in Stratos Datacenter.<br><br>Create a user named `rose` in App Server 1 without a home directory.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `man useradd`

## Steps

```bash
# Connec to the server
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Create the user without an interactive shell
useradd --no-create-home rose

# Check that the user was created
fgrep rose /etc/passwd
```

```
rose:x:1002:1002::/home/rose:/bin/bash
```

```bash
# A contents of /etc/passwd suggest a home directory exists, lets check that
ls -Ahl --color /home
```

```
total 8.0K
drwx------ 1 ansible ansible 4.0K Oct 15  2019 ansible
drwx------ 1 tony    tony    4.0K Jan 25  2020 tony
```

There is no `/home/rose` so the user was created without a home directory even though `/etc/passwd` suggests there is one.