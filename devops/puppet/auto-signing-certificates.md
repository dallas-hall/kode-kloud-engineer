# Automatic Signing Certificates

## Task

> During last weekly meeting, the Nautilus DevOps team has decided to use Puppet autosign config to auto sign the certificates for all Puppet agent nodes that they will keep adding under the Puppet master in Stratos DC. The Puppet master and CA servers are currently running on jump host and all three app servers are configured as Puppet agents. To set up autosign configuration on the Puppet master server, some configuration settings must be done. Please find below more details:<br><br>The Puppet server package is already installed on puppet master i.e jump server and the Puppet agent package is already installed on all App Servers. However, you may need to start the required services on all of these servers.<br>Configure autosign configuration on the Puppet master i.e jump server (by creating an `autosign.conf` in the puppet configuration directory) and assign the certificates for master node as well as for the all agent nodes. Use the respective host's FDQN to assign the certificates.<br>Use alias puppet (dns_alt_names) for master node and add its entry in /etc/hosts config file on master i.e Jump Server as well as on the all agent nodes i.e App Servers.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create a default intermediate certificate authority (CA) on the puppet master (jump host).
  * https://puppet.com/docs/puppet/latest/ssl_certificates.html
  * https://puppet.com/docs/puppet/latest/puppet_server_ca_cli.html
* Autosign is on by default but the whitelist is empty, populate this file.
  * https://puppet.com/docs/puppet/latest/config_file_autosign.html
  * https://puppet.com/docs/puppet/latest/ssl_autosign.html#ssl_basic_autosigning
* Where the configuration folder is.
  * https://puppet.com/docs/puppet/latest/dirs_confdir.html
* Get the local machine's configuration from the master.
  * https://puppet.com/docs/puppet/latest/man/agent.html

## Steps

```bash
# Check puppet is in our PATH on master
echo $PATH | grep puppet
```

```
/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/home/thor/.local/bin:/home/thor/bin
```

```bash
# Switch to root
sudo su

# Setup CA
puppetserver ca setup
```

```
Error:
Existing file at '/etc/puppetlabs/puppet/ssl/ca/ca_crt.pem'
Existing file at '/etc/puppetlabs/puppet/ssl/ca/ca_crl.pem'
Existing file at '/etc/puppetlabs/puppet/ssl/ca/infra_crl.pem'
Existing file at '/etc/puppetlabs/puppet/ssl/certs/jump_host.stratos.xfusioncorp.com.pem'
Existing file at '/etc/puppetlabs/puppet/ssl/certs/ca.pem'
Existing file at '/etc/puppetlabs/puppet/ssl/crl.pem'
Existing file at '/etc/puppetlabs/puppet/ssl/ca/ca_pub.pem'
Existing file at '/etc/puppetlabs/puppet/ssl/ca/inventory.txt'
Existing file at '/etc/puppetlabs/puppet/ssl/ca/infra_inventory.txt'
Existing file at '/etc/puppetlabs/puppet/ssl/ca/infra_serials'
Existing file at '/etc/puppetlabs/puppet/ssl/ca/serial'
Existing file at '/etc/puppetlabs/puppet/ssl/ca/signed/jump_host.stratos.xfusioncorp.com.pem'
Existing file at '/etc/puppetlabs/puppet/ssl/ca/root_key.pem'
Existing file at '/etc/puppetlabs/puppet/ssl/ca/ca_key.pem'
If you would really like to replace your CA, please delete the existing files first.
Note that any certificates that were issued by this CA will become invalid if you
replace it!
```

We need to do this another way

```bash
# Update DNS
cat >> /etc/hosts
```

```
127.0.0.1 localhost puppet
```

Close the file with `^D`

```bash
# Start Puppet
puppet resource service puppetserver ensure=running
```

```
service { 'puppetserver':
  ensure   => 'running',
  provider => 'systemd',
}
```

```bash
# Enable Puppet
puppet resource service puppetserver ensure=running
```

```
Notice: /Service[puppetserver]/enable: enable changed 'false' to 'true'
service { 'puppetserver':
  enable   => 'true',
  provider => 'systemd',
}
```

```bash
# Check Puppet is running
ss -lntp | grep 8140
```

```
LISTEN     0      50           *:8140                     *:*                   users:(("java",pid=13170,fd=38))
```

```bash
# Create the autosign configuration
cat > /etc/puppetlabs/puppet/autosign.conf
```

```
jump_host.stratos.xfusioncorp.com
stapp01.stratos.xfusioncorp.com
stapp02.stratos.xfusioncorp.com
stapp03.stratos.xfusioncorp.com
```

Close the file with `^D`

```bash
# Add some stuff to the master config file
vi /etc/puppetlabs/puppet/puppet.conf
```

```
certname = puppet
dns_alt_names = puppet,jump_host.stratos.xfusioncorp.com
server = jump_host.stratos.xfusioncorp.com
autosign = true
```

Close the file with `:x`

```bash
# Connect to application servers which are the puppet agents
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

Apply these steps to all 3 app nodes.

```bash
# Update DNS to point to the Puppet master server
cat >> /etc/hosts
```

```
172.16.238.3    jump_host.stratos.xfusioncorp.com puppet
```

Close the file with `^D`

```bash
# Register agents
cat >> /etc/puppetlabs/puppet/puppet.conf
```

```
server = puppet
```

Close the file with `^D`

```bash
# Get the local machine's configuration from the master
puppet agent -t
```

```
nfo: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Caching catalog for stapp02.stratos.xfusioncorp.com
Info: Applying configuration version '1685232270'
Notice: Applied catalog in 0.10 seconds
```

Check the certificates from the master.

```bash
# Check the certificates
puppetserver ca list --all # Can see all hosts, done
```

```
Signed Certificates:
    964369dd618d.c.argo-prod-us-east1.internal       (SHA256)  8C:19:47:FD:59:97:CE:0C:1D:DF:52:92:A3:BA:41:22:F2:3D:95:D4:38:D9:EF:85:65:9A:7B:B5:59:D5:CA:E6  alt names: ["DNS:puppet", "DNS:964369dd618d.c.argo-prod-us-east1.internal"]        authorization extensions: [pp_cli_auth: true]
    jump_host.stratos.xfusioncorp.com                (SHA256)  60:B5:45:40:E0:95:8C:B9:A3:5A:BF:55:B3:35:ED:FB:6C:1A:09:54:41:AA:4B:E6:2E:FE:4F:C6:9A:0F:79:89  alt names: ["DNS:puppet", "DNS:jump_host.stratos.xfusioncorp.com"] authorization extensions: [pp_cli_auth: true]
    stapp02.stratos.xfusioncorp.com                  (SHA256)  E5:B7:2C:A8:F0:60:C9:31:11:2F:DE:67:FB:72:48:6D:42:F9:B5:44:A6:CC:88:E8:07:1A:47:A9:2A:5B:E2:A9  alt names: ["DNS:stapp02.stratos.xfusioncorp.com"]
    stapp03.stratos.xfusioncorp.com                  (SHA256)  00:E2:60:B1:40:02:13:02:4B:FB:65:96:EF:F4:89:E4:9A:F1:DA:D6:64:3A:F7:7E:91:40:19:DD:0F:11:31:AE  alt names: ["DNS:stapp03.stratos.xfusioncorp.com"]
    stapp01.stratos.xfusioncorp.com                  (SHA256)  7D:E8:7C:81:D6:6D:E9:10:22:9B:31:79:C3:DE:CA:81:88:98:1F:A9:DF:80:85:DF:7F:E3:01:EA:C6:77:47:B8  alt names: ["DNS:stapp01.stratos.xfusioncorp.com"]
```

We are done.
