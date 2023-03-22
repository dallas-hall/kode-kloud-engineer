# String Manipulation

## Task

> There is some data on App Server 2 in Stratos DC. The Nautilus development team shared some requirement with the DevOps team to alter some of the data as per recent changes. The DevOps team is working to prepare a Puppet programming file to accomplish this. Below you can find more details about the task.<br><br>Create a Puppet programming file `demo.pp` under `/etc/puppetlabs/code/environments/production/manifests` directory on Puppet master node i.e Jump Server and by using puppet file_line resource perform the following tasks.<br>We have a file `/opt/sysops/demo.txt` on App Server 2. Use the Puppet programming file mentioned above to replace line `Welcome to Nautilus Industries!` to `Welcome to xFusionCorp Industries!`, no other data should be altered in this file.<br>Notes: :- Please make sure to run the puppet agent test using sudo on agent nodes, otherwise you can face certificate issues. In that case you will have to clean the certificates first and then you will be able to run the puppet agent test.<br>:- Before clicking on the Check button please make sure to verify puppet server and puppet agent services are up and running on the respective servers, also please make sure to run puppet agent test to apply/test the changes manually first.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Repalce strings - https://doc.wikimedia.org/puppet/puppet_types/file_line.html

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

# Create the expected file
cat > /etc/puppetlabs/code/environments/production/manifests/demo.pp
```

```
file_line { 'Update some text file.':
  ensure => present,
  path  => '/opt/sysops/demo.txt',
  match => 'Welcome to Nautilus Industries!',
  line => 'Welcome to xFusionCorp Industries!'
}
```

```bash
# Connect to application server which are the puppet agents
ssh steve@stapp02

# Switch to root
sudo -i

# Create a backup
cp /opt/sysops/demo.txt /opt/sysops/demo.txt.bak

# Get configuration changes
puppet agent -t
```

```
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Loading facts
Info: Caching catalog for stapp02.stratos.xfusioncorp.com
Info: Applying configuration version '1672530942'
Notice: /Stage[main]/Main/File_line[Update some text file.]/ensure: created
Notice: Applied catalog in 0.09 seconds
```

```bash
# Check the changes
cat /opt/sysops/demo.txt
```

```
This is  Nautilus sample file, created using Puppet!
Welcome to xFusionCorp Industries!
Please do not modify this file manually!
```

Changes have been made.