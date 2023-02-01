# File Permissions

## Task

> The Nautilus DevOps team has put some data on all app servers in Stratos DC. jump host is configured as Puppet master server, and all app servers are already been configured as Puppet agent nodes. The team needs to update the content of some of the exiting files, as well as need to update their permissions etc. Please find below more details about the task:<br><br>Create a Puppet programming file `news.pp` under `/etc/puppetlabs/code/environments/production/manifests` directory on the master node i.e Jump Server. Using puppet file resource, perform the below mentioned tasks.<br><br>A file named `ecommerce.txt` already exists under `/opt/data` directory on App Server 1.<br><br>Add content `Welcome to xFusionCorp Industries!` in `ecommerce.txt` file on App Server 1. Set its permissions to `0644`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Files -  https://coderwall.com/p/yfnesa/create-a-symlink-in-puppet &  https://puppet.com/docs/puppet/latest/types/file.html

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
sudo -i

# Create the file
cat > /etc/puppetlabs/code/environments/production/manifests/news.pp
```

```
file { '/opt/data/ecommerce.txt':
  content=> 'Welcome to xFusionCorp Industries!',
  mode=> '0644'
}
```

```bash
# Log into app server
ssh tony@stapp01

# Switch to root
sudo -i

# Pull the changes
puppet agent -t
```

```
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Caching catalog for stapp01.stratos.xfusioncorp.com
Info: Applying configuration version '1675249270'
Notice: /Stage[main]/Main/File[/opt/data/ecommerce.txt]/content:
--- /opt/data/ecommerce.txt     2023-02-01 10:57:37.994645575 +0000
+++ /tmp/puppet-file20230201-515-1mgwvy4        2023-02-01 11:01:11.896744473 +0000
@@ -0,0 +1 @@
+Welcome to xFusionCorp Industries!
\ No newline at end of file

Info: Computing checksum on file /opt/data/ecommerce.txt
Info: /Stage[main]/Main/File[/opt/data/ecommerce.txt]: Filebucketed /opt/data/ecommerce.txt to puppet with sum d41d8cd98f00b204e9800998ecf8427e
Notice: /Stage[main]/Main/File[/opt/data/ecommerce.txt]/content: content changed '{md5}d41d8cd98f00b204e9800998ecf8427e' to '{md5}b899e8a90bbb38276f6a00012e1956fe'
Notice: Applied catalog in 0.13 seconds
```

```bash
# Check the changes
ll /opt/data/ecommerce.txt
```

```
-rw-r--r-- 1 root root 34 Feb  1 11:01 /opt/data/ecommerce.txt
```

```bash
# Check the changes
cat /opt/data/ecommerce.txt
```

```
Welcome to xFusionCorp Industries!
```

Everything ok, we are done.
