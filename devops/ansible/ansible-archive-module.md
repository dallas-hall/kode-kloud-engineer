# Ansible Archive Module

## Task

> The Nautilus DevOps team has some data on each app server in Stratos DC that they want to copy to a different location. However, they want to create an archive of the data first, then they want to copy the same to a different location on the respective app server. Additionally, there are some specific requirements for each server. Perform the task using Ansible playbook as per requirements mentioned below:
>
> 1. Create a playbook named `playbook.yml` under `/home/thor/ansible` directory on jump host, an inventory file is already placed under `/home/thor/ansible/` directory on Jump Server itself.
> 2. Create an archive `blog.tar.gz`` (make sure archive format is `tar.gz`) of `/usr/src/itadmin/` directory ( present on each app server ) and copy it to `/opt/itadmin/` directory on all app servers. The user and group owner of archive `blog.tar.gz` should be `tony` for App Server 1, `steve` for App Server 2 and `banner` for App Server 3.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Playbooks.
  * https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html
* Archive.
  * https://docs.ansible.com/ansible/latest/collections/community/general/archive_module.html
* Connection variables.
  * https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#connection-variables

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
# Change to our working directory and checkout the inventory
cd ansible
cat inventory
```

```
stapp01 ansible_host=172.16.238.10 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=172.16.238.11 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=172.16.238.12 ansible_ssh_pass=BigGr33n ansible_user=banner
```

```bash
# Create the playbook
cat > playbook.yml
```

```yaml
---
- name: App servers work.
  hosts: all
  become: true
  become_user: root

  tasks:
  - name: Compress a folder.
    archive:
      path: /usr/src/itadmin/
      dest: /opt/itadmin/blog.tar.gz
      owner: "{{ ansible_user }}"
      group: "{{ ansible_user }}"
```

Close the file with control + d i.e. `^D`

```bash
# Run the playbook
ansible-playbook -i inventory playbook.yml
```

```
...
PLAY RECAP ****************************************************************************************************************************************************************
stapp01                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp02                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp03                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```bash
# Check the playbook work
ansible -i inventory all -m shell -a 'ls -Ahl /opt/itadmin'
```

```
stapp02 | CHANGED | rc=0 >>
total 4.0K
-rw-r--r-- 1 steve steve 174 Sep 18 05:17 blog.tar.gz
stapp03 | CHANGED | rc=0 >>
total 4.0K
-rw-r--r-- 1 banner banner 162 Sep 18 05:17 blog.tar.gz
stapp01 | CHANGED | rc=0 >>
total 4.0K
-rw-r--r-- 1 tony tony 165 Sep 18 05:17 blog.tar.gz
```

Looks good, we are done.
