# Troubleshooting Failed httpd Process

## Task

> While troubleshooting one of the issues on app servers in Stratos Datacenter DevOps team identified the root cause that the time isn't synchronized properly among the all app servers which causes issues sometimes. So team has decided to use a specific time server for all app servers, so that they all remain in sync. This task needs to be done using Puppet so as per details mentioned below please compete the task:<br><br>Create a puppet programming file demo.pp under `/etc/puppetlabs/code/environments/production/manifests` directory on puppet master node i.e on Jump Server. Within the programming file define a custom class `ntpconfig` to install and configure ntp server on app server 2.<br>Add NTP Server server `2.cn.pool.ntp.org` in default configuration file on app server 2, also remember to use iburst option for faster synchronization at startup.<br>Please note that do not try to start/restart/stop ntp service, as we already have a scheduled restart for this service tonight and we don't want these changes to be applied right now.<br><br>**Notes:** Please make sure to run the puppet agent test using sudo on agent nodes, otherwise you can face certificate issues. In that case you will have to clean the certificates first and then you will be able to run the puppet agent test.<br>Before clicking on the Check button please make sure to verify puppet server and puppet agent services are up and running on the respective servers, also please make sure to run puppet agent test to apply/test the changes manually first.<br>Please note that once lab is loaded, the puppet server service should start automatically on puppet master server, however it can take upto 2-3 minutes to start.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Install the NTP module.
  * https://puppet.com/docs/puppet/latest/quick_start_ntp.html
* Create necessary file.
  * https://puppet.com/docs/puppet/latest/quick_start_ntp.html
* Create custom class.
  * https://fullstack-puppet-docs.readthedocs.io/en/latest/puppet_modules.html
* Configuring iburst.
  * https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s2_configuring_the_iburst_option

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

# Install the NTP module
puppet module install puppetlabs-ntp
```

```
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└─┬ puppetlabs-ntp (v9.2.0)
  └── puppetlabs-stdlib (v8.5.0)
```


```bash
# Create our custom profile for NTP
cat > /etc/puppetlabs/code/environments/production/manifests/demo.pp
```

```
class ntpconfig {
        package { 'ntp':
                ensure=> 'present'
        }

        file { '/etc/ntp.conf':
                ensure=> 'present',
                content=> "server 2.cn.pool.ntp.org iburst\n"
  }
}
include ntpconfig
```

Close the file with control + d i.e. `^D`

```bash
# Connect to application server which are the puppet agents
ssh steve@stapp02

# Switch to root
sudo su

# Get configuration changes
puppet agent -t
```

```
...
Notice: /Stage[main]/Ntpconfig/File[/etc/ntp.conf]/content: content changed '{md5}dc9e5754ad2bb6f6c32b954c04431d0a' to '{md5}5db5a47d6bcaea4fa03bdec8068b5267'
Notice: Applied catalog in 18.63 seconds
```

Our configuration file has been applied.

```bash
# Check ntp configuration file
cat /etc/ntp.conf
```

```
server 2.cn.pool.ntp.org iburst
```

Our configuration file has been applied.

```bash
# Check puppet service
puppet resource service ntpd
```

```
service { 'ntpd':
  ensure   => 'stopped',
  enable   => 'false',
  provider => 'systemd',
}
```

The service is ready to be started later.
