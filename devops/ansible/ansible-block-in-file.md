# Ansible Block In File Module

## Task

> The Nautilus DevOps team wants to install and set up a simple `httpd` web server on all app servers in Stratos DC. Additionally, they want to deploy a sample web page for now using Ansible only. Therefore, write the required playbook to complete this task. Find more details about the task below.<br><br>We already have an `inventory` file under `/home/thor/ansible directory` on jump host. Create a `playbook.yml` under `/home/thor/ansible directory` on jump host itself.<br>Using the playbook, install `httpd` web server on all app servers. Additionally, make sure its service should up and running.<br>Using blockinfile Ansible module add some content in `/var/www/html/index.html` file. Below is the content:

```
Welcome to XfusionCorp!

This is Nautilus sample file, created using Ansible!

Please do not modify this file manually!
```

> The `/var/www/html/index.html` file's user and group owner should be `apache` on all app servers.<br>The `/var/www/html/index.html` file's permissions should be `0777` on all app servers.<br>Note:<br><br>i. Validation will try to run the playbook using command ansible-playbook -i inventory playbook.yml so please make sure the playbook works this way without passing any extra arguments.<br>ii. Do not use any custom or empty marker for blockinfile module.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Playbooks - https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html
* Block in file - https://docs.ansible.com/ansible/latest/collections/ansible/builtin/blockinfile_module.html
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
- name: Install & configure httpd.
  hosts: all
  become: yes
  tasks:
    - name: Install httpd
      # https://docs.ansible.com/ansible/latest/modules/package_module.html
      package:
        name: httpd
        state: present

    - name: Start service httpd, if not started
      service:
        name: httpd
        state: started

    - name: Create index.html
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache
        mode: '0777'
        state: touch

    - name : Add content to index.html
      blockinfile:
        path: /var/www/html/index.html
        block: |2
          Welcome to XfusionCorp!

          This is Nautilus sample file, created using Ansible!

          Please do not modify this file manually!
```

Close the file with control + d i.e. `^D`

```bash
# Run the playbook
ansible-playbook -i inventory playbook.yml
```

```
...
PLAY RECAP ***********************************************************************
stapp01                    : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
stapp02                    : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
stapp03                    : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

```bash
# Check httpd is running
ansible -i inventory all -m shell -a 'systemctl status httpd'
```

```
...
stapp02 | CHANGED | rc=0 >>
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2023-01-19 22:43:54 UTC; 1min 19s ago
...
stapp03 | CHANGED | rc=0 >>
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2023-01-19 22:43:54 UTC; 1min 19s ago
...
stapp01 | CHANGED | rc=0 >>
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2023-01-19 22:43:54 UTC; 1min 19s ago
...
```

They are all running.

```bash
# Check the file permissions are correct
ansible -i inventory all -m shell -a 'ls -Ahl /var/www/html'
```

```
stapp01 | CHANGED | rc=0 >>
total 4.0K
-rwxrwxrwx 1 apache apache 177 Jan 19 22:43 index.html
stapp02 | CHANGED | rc=0 >>
total 4.0K
-rwxrwxrwx 1 apache apache 177 Jan 19 22:43 index.html
stapp03 | CHANGED | rc=0 >>
total 4.0K
-rwxrwxrwx 1 apache apache 177 Jan 19 22:43 index.html
```

Permissions are correct.


```bash
# Check the web page is displayed correctly
curl stapp01
curl stapp02
curl stapp03
```

```
# BEGIN ANSIBLE MANAGED BLOCK
Welcome to XfusionCorp!

This is Nautilus sample file, created using Ansible!

Please do not modify this file manually!
# END ANSIBLE MANAGED BLOCK
```

All of them returned the same correct result, we are done.