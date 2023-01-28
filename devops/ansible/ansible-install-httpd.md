# Install httpd

## Task

> The Nautilus DevOps team is trying to setup a simple Apache web server on all app servers in Stratos DC using Ansible. They also want to create a sample html page for now with some app specific data on it. Below you can find more details about the task.<br><br>You will find a valid inventory file `/home/thor/playbooks/inventory` on jump host (which we are using as an Ansible controller).<br>Create a playbook `index.yml` under `/home/thor/playbooks` directory on jump host. Using `blockinfile` Ansible module create a file `facts.txt` under `/root` directory on all app servers and add the following given block in it. You will need to enable facts gathering for this task.<br>Ansible managed node IP is <default ipv4 address><br>(You can obtain default ipv4 address from Ansible's gathered facts by using the correct Ansible variable while taking into account Jinja2 syntax)<br>Install `httpd` server on all apps. After that make a copy of `facts.txt` file as `index.html` under `/var/www/html` directory. Make sure to start `httpd` service after that.<br>Note: Do not create a separate role for this task, just add all of the changes in index.yml playbook.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Playbooks - https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html
* Create file - https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html
* Host Variables - https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html
* Gather Facts - https://docs.ansible.com/ansible/latest/collections/ansible/builtin/gather_facts_module.html & https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html
* Copy - https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html


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
cd playbooks
cat inventory
```

```
stapp01 ansible_host=172.16.238.10 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=172.16.238.11 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=172.16.238.12 ansible_ssh_pass=BigGr33n ansible_user=banner
```

```bash
# Create the playbook
cat > index.yml
```

```yaml
# YAML format
---
- name: Install & configure httpd.
  hosts: all
  gather_facts: true
  become: yes
  tasks:
    - name: Create /root/facts.txt
      ansible.builtin.file:
        path: /root/facts.txt
        owner: root
        group: root
        mode: '0644'
        state: touch

    - name : Add gathered facts to /root/facts.txt
      blockinfile:
        path: /root/facts.txt
        block: |2
          Ansible managed node IP is {{ ansible_facts['default_ipv4']['address'] }}

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
        mode: '0744'
        state: touch

    - name : Copy facts.txt to index.yaml
      ansible.builtin.copy:
        src: /root/facts.txt
        dest: /var/www/html/index.html
        owner: apache
        group: apache
        mode: '0744'

```

```bash
# Run the playbook
ansible-playbook -i inventory index.yml
```

```
...
PLAY RECAP ***********************************************************************
stapp01                    : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
stapp02                    : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
stapp03                    : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

```bash
# Check the results
curl stapp01; curl stapp02; curl stapp03
```

```
# BEGIN ANSIBLE MANAGED BLOCK
Ansible managed node IP is 172.16.238.10
# END ANSIBLE MANAGED BLOCK
# BEGIN ANSIBLE MANAGED BLOCK
Ansible managed node IP is 172.16.238.11
# END ANSIBLE MANAGED BLOCK
# BEGIN ANSIBLE MANAGED BLOCK
Ansible managed node IP is 172.16.238.12
# END ANSIBLE MANAGED BLOCK
```

We are done.
