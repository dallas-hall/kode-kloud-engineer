# Symlinking

## Task

> Some directory structure in the Stratos Datacenter needs to be changed, there is a directory that needs to be linked to the default Apache document root. We need to accomplish this task using Puppet, as per the instructions given below:<br><br>Create a puppet programming file `cluster.pp` under `/etc/puppetlabs/code/environments/production/manifests` directory on puppet master node i.e on Jump Server. Within that define a class symlink and perform below mentioned tasks:<br>Create a symbolic link through puppet programming code. The source path should be `/opt/security` and destination path should be `/var/www/html` on Puppet agents 3 i.e on App Servers 3.<br>Create a blank file `blog.txt` under `/opt/security` directory on puppet agent 3 nodes i.e on App Servers 3.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Symlinking.
  * https://coderwall.com/p/yfnesa/create-a-symlink-in-puppet
* Files.
  * https://puppet.com/docs/puppet/latest/types/file.html

## Steps

```bash
# Check puppet is in our PATH
echo $PATH | grep puppet
```

```
/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/home/thor/.local/bin:/home/thor/bin
```

```bash
# Switch to root
sudo su

# Create the expected file
cat > /etc/puppetlabs/code/environments/production/manifests/cluster.pp
```

```
class symlink {
  file { '/opt/security':
    ensure=> 'link',
    target=> "/var/www/html"
  }
}
include symlink
```

Close the file `^D`

```bash
# Connect to application servers which are the puppet agents
ssh banner@stapp03

# Switch to root
sudo su

# Get configuration changes
puppet agent -t

# Create a test file
touch /opt/security/blog.txt

# Check the file exists through the symlink
ls -Ahl --color /var/www/html
```

```
-rw-r--r-- 1 root root 0 May 24 09:33 blog.txt
```

We are done.
