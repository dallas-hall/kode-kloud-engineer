# Ansible Install Samba Package

## Task

> The Nautilus Application development team wanted to test some applications on app servers in Stratos Datacenter. They shared some pre-requisites with the DevOps team, and packages need to be installed on app servers. Since we are already using Ansible for automating such tasks, please perform this task using Ansible as per details mentioned below:
>
> 1. Create an inventory file `/home/thor/playbook/inventory` on jump host and add all app servers in it.
> 2. Create an Ansible playbook `/home/thor/playbook/playbook.yml` to install `samba` package on all app servers using Ansible `yum` module.
> 3. Make sure user `thor` should be able to run the playbook on jump host.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Install packages with `yum`.
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html

## Steps

Follow [Passwordless ssh setup](../../linux-system-administrator/networking/passwordless-ssh-access.md) for every user.

```bash
# Check Ansible is install
ansible --version
```

```
ansible [core 2.15.0]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/thor/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.11/site-packages/ansible
  ansible collection location = /home/thor/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.11.4 (main, Jun 29 2023, 13:19:58) [GCC 8.5.0 20210514 (Red Hat 8.5.0-20)] (/usr/bin/python3.11)
  jinja version = 3.1.2
  libyaml = True
```

```bash
# Create the inventory.
cat > ~/playbook/inventory
```

```
stapp01 ansible_host=172.16.238.10 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=172.16.238.11 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=172.16.238.12 ansible_ssh_pass=BigGr33n ansible_user=banner
```

Close the file with control + d i.e. `^D`

```bash
# Create the playbook.
cat > ~/playbook/playbook.yml
```

```yaml
---
- name: Install samba using the yum module.
  hosts: all
  gather_facts: true
  become: true
  tasks:
    - name: Install samba.
      yum:
        name: samba
        state: present
```

```
# Run the playbook.
ansible-playbook -i ~/playbook/inventory ~/playbook/playbook.yml
```

```
...
PLAY RECAP *************************************************************************************************************************************
stapp01                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp02                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp03                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

We are done.
