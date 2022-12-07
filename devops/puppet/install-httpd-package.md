# Install httpd On App Server 2

## Task

> Some new packages need to be installed on app server 2 in Stratos Datacenter. The Nautilus DevOps team has decided to install the same using Puppet. Since jump host is already configured to run as Puppet master server and all app servers are already configured to work as the puppet agent nodes, we need to create required manifests on the Puppet master server so that the same can be applied on all Puppet agent nodes. Please find more details about the task below.<br><br>Create a Puppet programming file `cluster.pp` under `/etc/puppetlabs/code/environments/production/manifests` directory on master node i.e Jump Server and using puppet package resource perform the tasks given below.<br>Install package `httpd` through Puppet package resource only on App server 2i.epuppet agent node 2`.<br><br>Notes: :- Please make sure to run the puppet agent test using sudo on agent nodes, otherwise you can face certificate issues. In that case you will have to clean the certificates first and then you will be able to run the puppet agent test.<br>:- Before clicking on the Check button please make sure to verify puppet server and puppet agent services are up and running on the respective servers, also please make sure to run puppet agent test to apply/test the changes manually first.<br>:- Please note that once lab is loaded, the puppet server service should start automatically on puppet master server, however it can take upto 2-3 minutes to start.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Install a package - https://www.puppetcookbook.com/posts/install-package.html & https://puppet.com/docs/puppet/7/types/package.html
* Nodes - https://puppet.com/docs/puppet/7/lang_node_definitions.html
* Classes - https://puppet.com/docs/puppet/7/lang_classes.html
* Packages - https://puppet.com/docs/puppet/7/types/package.html

## Steps

```bash
# Switch to root
sudo -i

# Check puppet is in our PATH
echo $PATH | grep puppet
```

```
/usr/local/sbin:/sbin:/bin:/usr/sbin:/usr/bin:/opt/puppetlabs/bin:/root/bin
```

```bash
# View all nodes
puppetserver ca list --all
```

```
Signed Certificates:
    964369dd618d.c.argo-prod-us-east1.internal       (SHA256)  8C:19:47:FD:59:97:CE:0C:1D:DF:52:92:A3:BA:41:22:F2:3D:95:D4:38:D9:EF:85:65:9A:7B:B5:59:D5:CA:E6  alt names: ["DNS:puppet", "DNS:964369dd618d.c.argo-prod-us-east1.internal"]        authorization extensions: [pp_cli_auth: true]
    jump_host.stratos.xfusioncorp.com                (SHA256)  E3:F9:23:5A:9F:85:17:07:3F:1E:75:E3:26:AC:23:6A:B7:31:9D:FE:92:78:9C:06:D7:C6:FB:07:26:0C:55:0E  alt names: ["DNS:puppet", "DNS:jump_host.stratos.xfusioncorp.com"] authorization extensions: [pp_cli_auth: true]
    stapp02.stratos.xfusioncorp.com                  (SHA256)  FC:FA:66:96:60:2F:24:30:2C:AD:4B:5E:5C:8F:A0:FA:09:2E:0F:88:5B:53:94:8C:CC:8D:E4:32:14:4D:7B:9B  alt names: ["DNS:stapp02.stratos.xfusioncorp.com"]
    stapp03.stratos.xfusioncorp.com                  (SHA256)  0C:A5:9A:D0:D9:F4:2F:19:8B:13:5C:1A:E4:65:2C:72:6D:D4:A1:03:9D:B6:D0:B3:31:8C:57:38:3C:5A:47:81  alt names: ["DNS:stapp03.stratos.xfusioncorp.com"]
    stapp01.stratos.xfusioncorp.com                  (SHA256)  53:C9:D9:44:34:B8:D7:4C:FF:41:C8:6E:B9:C7:2D:C8:6B:9A:B9:8F:C6:10:43:6C:6C:83:FB:F9:4E:25:AC:FD  alt names: ["DNS:stapp01.stratos.xfusioncorp.com"]
```

```bash
# Create our install file
cat > /etc/puppetlabs/code/environments/production/manifests/cluster.pp
```

```
class my_httpd {
  package { 'httpd':
    ensure => 'installed',
  }
}

node 'stapp02.stratos.xfusioncorp.com' {
  include my_httpd
}
```

Close the file with control + d i.e. `^D`


```bash
# Connect to app server & gain root access.
ssh steve@stapp02
sudo -i

# Apply the puppet configuration on this node
puppet agent -t
```

```
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Caching catalog for stapp02.stratos.xfusioncorp.com
Info: Applying configuration version '1670373550'
Notice: /Stage[main]/My_httpd/Package[httpd]/ensure: created
Info: Creating state file /opt/puppetlabs/puppet/cache/state/state.yaml
Notice: Applied catalog in 7.19 seconds
```

We are done.
