# Ansible Block In File Module

## Task

> The Nautilus DevOps team wants to install and set up a simple httpd web server on all app servers in Stratos DC. Additionally, they want to deploy a sample web page for now using Ansible only. Therefore, write the required playbook to complete this task. Find more details about the task below.
>
> We already have an `inventory` file under `/home/thor/ansible` directory on jump host. Create a `playbook.yml` under `/home/thor/ansible` directory on jump host itself.
>
> 1. Using the `playbook`, install `httpd` web server on all app servers. Additionally, make sure its service should up and running.
> 2. Using `blockinfile` Ansible module add some content in `/var/www/html/index.html` file. Below is the content:

```
Welcome to XfusionCorp!

This is Nautilus sample file, created using Ansible!

Please do not modify this file manually!
```
> 3. The `/var/www/html/index.html` file's user and group owner should be `apache` on all app servers.
> 4. The `/var/www/html/index.html` file's permissions should be `0755` on all app servers.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Playbooks.
  * https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html
* Block in file.
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/blockinfile_module.html
* YAML block chomping.
  * https://yaml-multiline.info/


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
- name: Install & configure httpd.
  hosts: all
  become: yes
  tasks:
    - name: Install httpd
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
        mode: '0755'
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
PLAY RECAP *************************************************************************************************************************************
stapp01                    : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp02                    : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp03                    : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```bash
# Check httpd is running
ansible -i inventory all -m shell -a 'systemctl status httpd'
```

```
stapp03 | CHANGED | rc=0 >>
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-09-18 05:58:06 UTC; 19s ago
...
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-09-18 05:58:06 UTC; 18s ago
...
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-09-18 05:58:06 UTC; 19s ago
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
-rwxr-xr-x 1 apache apache 178 Sep 18 05:58 index.html
stapp02 | CHANGED | rc=0 >>
total 4.0K
-rwxr-xr-x 1 apache apache 178 Sep 18 05:58 index.html
stapp03 | CHANGED | rc=0 >>
total 4.0K
-rwxr-xr-x 1 apache apache 178 Sep 18 05:58 index.html
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
