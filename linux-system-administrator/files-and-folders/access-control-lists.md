# Access Control Lists (ACL)s

## Task

> The Nautilus security team performed an audit on all servers present in Stratos DC. During the audit some critical data/files were identified which were having the wrong permissions as per security standards. Once the report was shared with the production support team, they started fixing the issues. It has been identified that one of the files named `/etc/hosts` on Nautilus App 1 server has wrong permissions, so that needs to be fixed and the correct ACLs needs to be set.<br><br>1. The user owner and group owner of the file should be `root` user.<br>2. Others must have read only permissions on the file.<br>3. User `javed` must not have any permission on the file.<br>4. User `jerome` should have read only permission on the file.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Linux ACLS
  * https://www.redhat.com/sysadmin/access-control-lists
* `man scp`

## Steps

```bash
# Connect to the first app server
ssh tony@stapp01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check the file permissions
ls -Ahl /etc/hosts
```

```
-rw-r--r-- 1 root root 374 Aug  7 02:56 /etc/hosts
```

```bash
# Check the ACLs
getfacl /etc/hosts
```

```
getfacl: Removing leading '/' from absolute path names
# file: etc/hosts
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

```bash
# Block javed via ACL
setfacl -m u:javed:--- /etc/hosts

# Allow jerome via ACL
setfacl -m u:jerome:r-- /etc/hosts

# Check the ACLs
[root@stapp01 ~]# getfacl /etc/hosts
```

```
getfacl: Removing leading '/' from absolute path names
# file: etc/hosts
# owner: root
# group: root
user::rw-
user:javed:---
user:jerome:r--
group::r--
mask::r--
other::r--
```

We are done.
