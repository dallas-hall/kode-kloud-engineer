# Ansible Copy Module

## Task

> One of the Nautilus DevOps team members was working on to test an Ansible playbook on jump host. However, he was only able to create the inventory, and due to other priorities that came in he has to work on other tasks. Please pick up this task from where he left off and complete it. Below are more details about the task:<br><br>The inventory file `/home/thor/ansible/inventory` seems to be having some issues, please fix them. The playbook needs to be run on App Server 2 in Stratos DC, so inventory file needs to be updated accordingly.<br>Create a playbook `/home/thor/ansible/playbook.yml` and add a task to create an empty file `/tmp/file.txt` on App Server 2.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* File module.
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html

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
stapp02 ansible_host=172.16.238.11 ansible_ssh_pass=Am3ric@ ansible_user=steve
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
    - name: Create file.
      file:
        path: /tmp/file.txt
        state: touch
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
stapp02                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Looks okay, but lets check.

```bash
# Check if file exists
ansible -i inventory all -m shell -a 'ls -Ahl /tmp/file.txt'
```

```
stapp02 | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 0 Jul 31 09:34 /tmp/file.txt
```

We are done.
