# Ansible File Module

## Task

> The Nautilus DevOps team is practicing some of the Ansible modules and creating and testing different Ansible playbooks to accomplish tasks. Recently they started testing an Ansible file module to create soft links on all app servers. Below you can find more details about it.
>
> Write a playbook.yml under `/home/thor/ansible` directory on jump host, an inventory file is already present under `/home/thor/ansible` directory on jump host itself. Using this playbook accomplish below given tasks:
>
> * Create an empty file `/opt/dba/blog.txt` on app server 1; its user owner and group owner should be `tony`. Create a symbolic link of source path `/opt/dba` to destination `/var/www/html`.
> * Create an empty file `/opt/dba/story.txt` on app server 2; its user owner and group owner should be `steve`. Create a symbolic link of source path `/opt/dba` to destination `/var/www/html`.
> * Create an empty file `/opt/dba/media.txt` on app server 3; its user owner and group owner should be `banner`. Create a symbolic link of source path `/opt/dba` to destination `/var/www/html`.
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
        mode: '0644'
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
    - name: Create symlink.
      file:
        src: /opt/dba
        dest: /var/www/html
        state: link
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"

- hosts: stapp02
  become: yes
  tasks:
    - name: Create file.
      file:
        path: /opt/dba/story.txt
        state: touch
        mode: '0644'
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
    - name: Create symlink.
      file:
        src: /opt/dba
        dest: /var/www/html
        state: link
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"

- hosts: stapp03
  become: yes
  tasks:
    - name: Create file.
      file:
        path: /opt/dba/media.txt
        state: touch
        mode: '0644'
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
    - name: Create symlink.
      file:
        src: /opt/dba
        dest: /var/www/html
        state: link
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"

```

Close the file with control + d i.e. `^D`

```bash
# Run playbook
cd playbook
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
ansible -i inventory all -m shell -a 'ls -Adhl /var/www/html && ls -Ahl /var/www/html/'
```

```
stapp01 | CHANGED | rc=0 >>
lrwxrwxrwx 1 root root 8 Nov 24 03:55 /var/www/html -> /opt/dba
total 0
-rw-r--r-- 1 tony tony 0 Nov 24 03:57 blog.txt
stapp03 | CHANGED | rc=0 >>
lrwxrwxrwx 1 root root 8 Nov 24 03:57 /var/www/html -> /opt/dba
total 0
-rw-r--r-- 1 banner banner 0 Nov 24 03:57 media.txt
stapp02 | CHANGED | rc=0 >>
lrwxrwxrwx 1 root root 8 Nov 24 03:57 /var/www/html -> /opt/dba
total 0
-rw-r--r-- 1 steve steve 0 Nov 24 03:57 story.txt
```

We are done.
