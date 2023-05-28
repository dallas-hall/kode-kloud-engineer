# Ansible Archive Module

## Task

> The Nautilus DevOps team has some data on jump host in Stratos DC that they want to copy on all app servers in the same data center. However, they want to create an archive of data and copy it to the app servers. Additionally, there are some specific requirements for each server. Perform the task using Ansible playbook as per requirements mentioned below:<br><br>Create a `playbook.yml` under `/home/thor/ansible` on jump host, an `inventory` file is already placed under `/home/thor/ansible/` on Jump Server itself.<br>Create an archive `cluster.tar.gz` (make sure archive format is tar.gz) of `/usr/src/data/` directory ( present on each app server ) and copy it to `/opt/data/` directory on all app servers. The user and group owner of archive `cluster.tar.gz` should be `tony` for App Server 1, `steve` for App Server 2 and `banner` for App Server 3.<br>Note: Validation will try to run playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure playbook works this way, without passing any extra arguments.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Playbooks.
  * https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html
* Archive.
  * https://docs.ansible.com/ansible/latest/collections/community/general/archive_module.html

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
      path: /usr/src/data/
      dest: /opt/data/cluster.tar.gz
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
ansible -i inventory all -m shell -a 'ls -l /opt/data'
```

```
stapp03 | CHANGED | rc=0 >>
total 4
-rw-r--r-- 1 banner banner 162 Jan  5 22:47 cluster.tar.gz
stapp02 | CHANGED | rc=0 >>
total 4
-rw-r--r-- 1 steve steve 174 Jan  5 22:47 cluster.tar.gz
stapp01 | CHANGED | rc=0 >>
total 4
-rw-r--r-- 1 tony tony 165 Jan  5 22:47 cluster.tar.gz
```

Looks good, we are done.
