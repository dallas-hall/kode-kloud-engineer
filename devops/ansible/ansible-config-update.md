# Ansible Block In File Module

## Task

> To manage all servers within the stack using Ansible, the Nautilus DevOps team is planning to use a common sudo user among all servers. Ansible will be able to use this to perform different tasks on each server. This is not finalized yet, but the team has decided to first perform testing. The DevOps team has already installed Ansible on jump host using `yum`, and they now have the following requirement:<br><br>On jump host make appropriate changes so that Ansible can use `yousuf` as a default ssh user for all hosts. Make changes in Ansible's default configuration only —please do not try to create a new config.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Config
  * https://docs.ansible.com/ansible/latest/reference_appendices/config.html
  * https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings
  * https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-remote-user

## Steps

Follow [Passwordless ssh setup](../../linux-system-administrator/networking/passwordless-ssh-access.md) for every user.

```bash
# Check Ansible is install
ansible --version
```

```
ansible 2.9.9
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.5 (default, Jun 20 2019, 20:27:34) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
```

```bash
# Switch to root
sudo -i

# Create a backup
cp /etc/ansible/ansible.cfg /etc/ansible/ansible.cfg.bak

# Search for the remote_user
grep remote_user /etc/ansible/ansible.cfg
```

```
#remote_user = root
```

```bash
# Set the default ssh user
sed -i -r 's/#remote_user = root/remote_user = yousuf/g' /etc/ansible/ansible.cfg

# Check the changes
grep remote_user /etc/ansible/ansible.cfg
```

```
remote_user = yousuf
```

We are done.