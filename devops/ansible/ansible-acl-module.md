# Ansible ACL Module

## Task

> There are some files that need to be created on all app servers in Stratos DC. The Nautilus DevOps team want these files to be owned by user `root` only however, they also want that the app specific user to have a set of permissions on these files. All tasks must be done using Ansible only, so they need to create a playbook. Below you can find more information about the task.
>
> Create a playbook named `playbook.yml` under `/home/thor/ansible` directory on jump host, an inventory file is already present under `/home/thor/ansible` directory on Jump Server itself.
>
> * Create an empty file `blog.txt` under `/opt/dba/` directory on app server 1. Set some acl properties for this file. Using acl provide `read '(r)'` permissions to group `tony` (i.e `entity` is tony and `etype` is group).
> * Create an empty file `story.txt` under `/opt/dba/` directory on app server 2. Set some acl properties for this file. Using acl provide `read + write '(rw)'` permissions to user `steve` (i.e `entity` is steve and `etype` is user).
> * Create an empty file `media.txt` under `/opt/dba/` on app server 3. Set some acl properties for this file. Using acl provide `read + write '(rw)'` permissions to group `banner` (i.e `entity` is banner and `etype` is group).
>
> **Note:** Validation will try to run the playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure playbook works this way without passing any extra arguments.


## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* File module.
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html
* Special variables
  * https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html
* Targetting hosts
  * https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html
* ACLs
  * https://docs.ansible.com/ansible/latest/collections/ansible/posix/acl_module.html

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
cat > ~/playbook/inventory
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
cat > ~/playbook/playbook.yml
```

```yaml
- hosts: stapp01
  become: yes
  tasks:
    - name: Create file.
      file:
        path: /opt/dba/blog.txt
        state: touch
        mode: '0600'
        owner: root
        group: root
    - name: Set ACLs
      acl:
        path: /opt/dba/blog.txt
        entity: "{{ ansible_user }}"
        etype: group
        permissions: r
        state: present

- hosts: stapp02
  become: yes
  tasks:
    - name: Create file.
      file:
        path: /opt/dba/story.txt
        state: touch
        mode: '0600'
        owner: root
        group: root
    - name: Set ACLs
      acl:
        path: /opt/dba/story.txt
        entity: "{{ ansible_user }}"
        etype: user
        permissions: rw
        state: present

- hosts: stapp03
  become: yes
  tasks:
    - name: Create file.
      file:
        path: /opt/dba/media.txt
        state: touch
        mode: '0600'
        owner: root
        group: root
    - name: Set ACLs
      acl:
        path: /opt/dba/media.txt
        entity: "{{ ansible_user }}"
        etype: group
        permissions: rw
        state: present

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
# Check the symlinks and the contents.
ansible -i inventory all -m shell -a 'ls -Ahl /opt/dba; getfacl /opt/dba/*' -f 1
```

```
stapp01 | CHANGED | rc=0 >>
total 0
-rw-r-----+ 1 root root 0 Nov 24 04:15 blog.txt
# file: opt/dba/blog.txt
# owner: root
# group: root
user::rw-
group::---
group:tony:r--
mask::r--
other::---getfacl: Removing leading '/' from absolute path names
stapp02 | CHANGED | rc=0 >>
total 0
-rw-rw----+ 1 root root 0 Nov 24 04:15 story.txt
# file: opt/dba/story.txt
# owner: root
# group: root
user::rw-
user:steve:rw-
group::---
mask::rw-
other::---getfacl: Removing leading '/' from absolute path names
stapp03 | CHANGED | rc=0 >>
total 0
-rw-rw----+ 1 root root 0 Nov 24 04:15 media.txt
# file: opt/dba/media.txt
# owner: root
# group: root
user::rw-
group::---
group:banner:rw-
mask::rw-
other::---getfacl: Removing leading '/' from absolute path names
```

We are done.
