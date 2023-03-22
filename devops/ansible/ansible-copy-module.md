# Ansible Copy Module

## Task

> There is data on jump host that needs to be copied on all application servers in Stratos DC. Nautilus DevOps team want to perform this task using Ansible. Perform the task as per details mentioned below:<br><br>a. On jump host create an inventory file `/home/thor/ansible/inventory` and add all application servers as managed nodes.<br>b. On jump host create a playbook `/home/thor/ansible/playbook.yml` to copy `/usr/src/data/index.html` file to all application servers at location `/opt/data`.
>
## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Copy module - https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html

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
# Create inventory
cat > /home/thor/ansible/inventory
```

```yaml
all:
  hosts:
    stapp01:
      ansible_user: tony
      ansible_password: Ir0nM@n
    stapp02:
      ansible_user: steve
      ansible_password: Am3ric@
    stapp03:
      ansible_user: banner
      ansible_password: BigGr33n
  vars:
    ansible_connection: ssh
```

Close the file with control + d i.e. `^D`

```bash
# Create the playbook
cat > /home/thor/ansible/playbook.yml
```

```yaml
- hosts: all
  become: yes
  tasks:
    - name: Copy file.
      copy:
        src: /usr/src/data/index.html
        dest: /opt/data
```

Close the file with control + d i.e. `^D`

```bash
# Run playbook
cd ansible
ansible-playbook -i inventory playbook.yml
```

```
...
PLAY RECAP ***********************************************************************
stapp01                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp02                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp03                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Looks okay, but lets check.

```bash
# Check if file exists
ansible -i inventory all -m shell -a 'ls /opt/data'
```

```
stapp02 | CHANGED | rc=0 >>
index.html
stapp03 | CHANGED | rc=0 >>
index.html
stapp01 | CHANGED | rc=0 >>
index.html
```

We are done.
