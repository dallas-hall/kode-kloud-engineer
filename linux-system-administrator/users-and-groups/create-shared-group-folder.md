# Create Shared Group Folder

## Task

> The Nautilus team doesn't want its data to be accessed by any of the other groups/teams due to security reasons and want their data to be strictly accessed by the `devops` group of the team.<br><br>Setup a collaborative directory `/devops/data` on Nautilus App 3 server in Stratos Datacenter.<br><br>The directory should be group owned by the group `devops` and the group should own the files inside the directory. The directory should be read/write/execute to the group owners, and others should not have any access.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Permissions
  * `man chmod`
  * `man chown`
  * https://www.redhat.com/sysadmin/suid-sgid-sticky-bit

## Steps

```bash
# Connect to application server
ssh banner@stapp03

# Check current Linux version, it was CentOS Steam 8.
cat /etc/*rel*

# Switch to root
sudo -i

# Check the group exists
grep devops /etc/group
```

```
devops:x:1002:
```

```bash
# Check the folder exists
ls -Ahl /devops
```

```
ls: cannot access /devops: No such file or directory
```

```bash
# Create the folders
mkdir -p /devops/data

# Change the group ownership for the folders.
chown -R :devops /devops

# Change permissions for the folders
chmod -R 770 /devops

# Set GUID for the folders
chmod -R g+s /devops

# Check ownership and permissions
ls -Adhl /devops /devops/data
```

```
drwxrws--- 3 root devops 4.0K Aug 31 07:54 /devops
drwxrws--- 2 root devops 4.0K Aug 31 07:52 /devops/data
```

Note the *s* in the groups execution bit, this is the GUID.

```bash
# Test the shared group setup.
touch /devops/test.txt
ls -Ahl /devops/test.txt
```

```
-rw-r--r-- 1 root devops 0 Aug 31 07:55 /devops/test.txt```

Even though we created the file as the *root* user, the owning group is *devops*. We are done.
