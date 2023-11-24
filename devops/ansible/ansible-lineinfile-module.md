# Ansible Line In File Module

## Task

> The Nautilus DevOps team want to install and set up a simple httpd web server on all app servers in Stratos DC. They also want to deploy a sample web page using Ansible. Therefore, write the required playbook to complete this task as per details mentioned below.
>
> We already have an inventory file under `/home/thor/ansible directory` on jump host. Write a playbook `playbook.yml` under `/home/thor/ansible directory` on jump host itself. Using the playbook perform below given tasks:
>
> * Install `httpd`` web server on all app servers, and make sure its service is up and running.
> * Create a file `/var/www/html/index.html` with content: `This is a Nautilus sample file, created using Ansible!`
> * Using `lineinfile` Ansible module add some more content in `/var/www/html/index.html` file. Below is the content: `Welcome to Nautilus Group!` Also make sure this new line is added at the top of the file.
> * The `/var/www/html/index.html` file's user and group owner should be `apache` on all app servers.
> * The `/var/www/html/index.html` file's permissions should be `0777` on all app servers.
>
> **Note:** Validation will try to run the playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure playbook works this way without passing any extra arguments.


## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Shell module
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html
* Lineinfile module
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html


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
- hosts: all
  become: yes
  tasks:
    - name: Install httpd
      package:
        name: httpd
        state: present

    - name: Start and enable the httpd service
      service:
        name: httpd
        state: started
        enabled: true
    
    - name: Create index.html
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache
        mode: '0777'
        state: touch

    - name: Create index.html content
      shell: echo 'This is a Nautilus sample file, created using Ansible!' > /var/www/html/index.html
      
    - name: Add new content line to index.html
      lineinfile:
        path: /var/www/html/index.html
        line: 'Welcome to Nautilus Group!'
        insertbefore: 'BOF'

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
# Check the service status.
ansible -i inventory all -m shell -a 'systemctl status httpd | head -n 3' -f 1
```

```
stapp01 | CHANGED | rc=0 >>
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-11-24 06:09:34 UTC; 42s ago
stapp02 | CHANGED | rc=0 >>
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-11-24 06:09:34 UTC; 42s ago
stapp03 | CHANGED | rc=0 >>
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-11-24 06:09:34 UTC; 43s ago
```

```bash
# Check the file content.
ansible -i inventory all -m shell -a 'ls -Ahl /var/www/html/index.html; cat /var/www/html/index.html' -f 1
```

```
stapp01 | CHANGED | rc=0 >>
-rwxrwxrwx 1 apache apache 82 Nov 24 06:09 /var/www/html/index.html
Welcome to Nautilus Group!
This is a Nautilus sample file, created using Ansible!
stapp02 | CHANGED | rc=0 >>
-rwxrwxrwx 1 apache apache 82 Nov 24 06:09 /var/www/html/index.html
Welcome to Nautilus Group!
This is a Nautilus sample file, created using Ansible!
stapp03 | CHANGED | rc=0 >>
-rwxrwxrwx 1 apache apache 82 Nov 24 06:09 /var/www/html/index.html
Welcome to Nautilus Group!
This is a Nautilus sample file, created using Ansible!
```

We are done.
