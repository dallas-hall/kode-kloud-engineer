# Create Shared Group Folder

## Task

> The Nautilus team doesn't want its data to be accessed by any of the other groups/teams due to security reasons and want their data to be strictly accessed by the `sysadmin` group of the team.<br><br>Setup a collaborative directory `/sysadmin/data` on Nautilus App 2 server in Stratos Datacenter.<br><br>The directory should be group owned by the group `sysadmin` and the group should own the files inside the directory. The directory should be read/write/execute to the group owners, and others should not have any access.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `man chmod`, `man chown`, and https://www.redhat.com/sysadmin/suid-sgid-sticky-bit

## Steps

```bash
# Connect to application server
ssh steve@stapp02

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Check the group exists
grep sysadmin /etc/group
```

```
sysadmin:x:1002:john
```

```bash
# Check the folder exists
ls -Ahl /sysadmin
```

```
ls: cannot access /sysadmin: No such file or directory
```

```bash
# Create the folders
mkdir -p /sysadmin/data

# Change ownership for the folders
chown -R :sysadmin /sysadmin

# Change permissions for the folders
chmod -R 770 /sysadmin

# Set GUID for the folders
chmod -R g+s /sysadmin

# Check ownership and permissions
ls -Adhl /sysadmin /sysadmin/data
```

```
drwxrws--- 3 root sysadmin 4.0K Aug 23 09:25 /sysadmin
drwxrws--- 2 root sysadmin 4.0K Aug 23 09:25 /sysadmin/data
```

Note the *s* in the groups execution bit, this is the GUID.

```
# Test the setup
cd /sysadmin
touch test
ls -Ahl test
```

```
drwxrws--- 2 root sysadmin 4.0K Aug 23 09:25 data
-rw-r--r-- 1 root sysadmin    0 Aug 23 09:28 test
```

Even though we created the file as *root* the owning group is *sysadmin*.