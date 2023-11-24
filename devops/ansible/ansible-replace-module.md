# Ansible Repalce Module

## Task

> There is some data on all app servers in Stratos DC. The Nautilus development team shared some requirement with the DevOps team to alter some of the data as per recent changes they made. The DevOps team is working to prepare an Ansible playbook to accomplish the same. Below you can find more details about the task.
>
> Write a `playbook.yml` under `/home/thor/ansible` on jump host, an inventory is already present under `/home/thor/ansible` directory on Jump host itself. Perform below given tasks using this playbook:
>
> * We have a file `/opt/data/blog.txt` on app server 1. Using Ansible replace module replace string `xFusionCorp` to `Nautilus` in that file.
> * We have a file `/opt/data/story.txt` on app server 2. Using Ansible replace module replace the string `Nautilus` to `KodeKloud` in that file.
> * We have a file `/opt/data/media.txt` on app server 3. Using Ansible replace module replace string `KodeKloud` to `xFusionCorp Industries` in that file.
>
> **Note:** Validation will try to run the playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure playbook works this way without passing any extra arguments.


## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Replace module
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/replace_module.html


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
# Create inventory
cat > ~/ansible/inventory
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
cat > ~/ansible/playbook.yml
```

```yaml
- hosts: stapp01
  become: yes
  tasks:
    - name: Regex replace file.
      replace:
        path: /opt/data/blog.txt
        regexp: 'xFusionCorp'
        replace: 'Nautilus'

- hosts: stapp02
  become: yes
  tasks:
    - name: Regex replace file.
      replace:
        path: /opt/data/story.txt
        regexp: 'Nautilus'
        replace: 'KodeKloud'

- hosts: stapp03
  become: yes
  tasks:
    - name: Regex replace file.
      replace:
        path: /opt/data/media.txt
        regexp: 'KodeKloud'
        replace: 'xFusionCorp Industries'
```

Close the file with control + d i.e. `^D`

```bash
# Run playbook
cd ansible
ansible-playbook -i inventory playbook.yml
```

```
...
PLAY RECAP *************************************************************************************************************************************
stapp01                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp02                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp03                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Looks okay, but lets check.

```bash
# Check the file contents.
ansible -i inventory all -m shell -a 'cat /opt/data/*' -f 1
```

```
stapp01 | CHANGED | rc=0 >>
Welcome to Nautilus Industries !
stapp02 | CHANGED | rc=0 >>
Welcome to KodeKloud Group !
stapp03 | CHANGED | rc=0 >>
Welcome to xFusionCorp Industries !
```

We are done.
