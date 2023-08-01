# Ansible File Module

## Task

> The Nautilus DevOps team is working to test several Ansible modules on servers in Stratos DC. Recently they wanted to test the file creation on remote hosts using Ansible. Find below more details about the task:<br><br>a. Create an inventory file `~/playbook/inventory` on jump host and add all app servers in it.<br>b. Create a playbook `~/playbook/playbook.yml` to create a blank file `/tmp/data.txt` on all app servers.<br>c. The `/tmp/data.txt` file permission must be `0755`.<br>d. The user/group owner of file `/tmp/data.txt` must be `tony` on app server 1, `steve` on app server 2 and `banner` on app server 3.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* File module.
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html
* Special variables
  * https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html

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
- hosts: all
  become: yes
  tasks:
    - name: Create file.
      file:
        path: /tmp/data.txt
        state: touch
        mode: '0755'
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
# Check if file exists
ansible -i inventory all -m shell -a 'ls -Ahl /tmp/data.txt'
```

```
stapp02 | CHANGED | rc=0 >>
-rwxr-xr-x 1 steve steve 0 Aug  1 08:53 /tmp/data.txt
stapp01 | CHANGED | rc=0 >>
-rwxr-xr-x 1 tony tony 0 Aug  1 08:53 /tmp/data.txt
stapp03 | CHANGED | rc=0 >>
-rwxr-xr-x 1 banner banner 0 Aug  1 08:53 /tmp/data.txt
```

We are done.
