# httpd Role With Jinja2 Templating

## Task

> One of the Nautilus DevOps team members is working on to develop a role for httpd installation and configuration. Work is almost completed, however there is a requirement to add a jinja2 template for index.html file. Additionally, the relevant task needs to be added inside the role. The inventory file `~/ansible/inventory` is already present on jump host that can be used. Complete the task as per details mentioned below:
> * Update `~/ansible/playbook.yml` playbook to run the `httpd` role on App Server 1.
> * Create a jinja2 template `index.html.j2` under `/home/thor/ansible/role/httpd/templates/` directory and add a line `This file was created using Ansible on <respective server>` (for example `This file was created using Ansible on stapp01` in case of App Server 1). Also please make sure not to hard code the server name inside the template. Instead, use `inventory_hostname` variable to fetch the correct value.
> * Add a task inside `/home/thor/ansible/role/httpd/tasks/main.yml` to copy this template on App Server 1 under `/var/www/html/index.html`. Also make sure that `/var/www/html/index.html` file's permissions are `0744`.
> * The user/group owner of `/var/www/html/index.html` file must be respective sudo user of the server (for example `tony` in case of stapp01).

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Special variables.
  * https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#term-inventory_hostname
* Templates.
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html#examples


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
# Update playbook.
cat > playbook.yml
```

```yaml
---
- hosts: stapp01
  become: yes
  become_user: root
  roles:
    - role/httpd
```

Close the file with control + d i.e. `^D`


```bash
# Update template.
cat > /home/thor/ansible/role/httpd/templates/index.html.j2
```

```
This file was created using Ansible on {{ inventory_hostname }}
```

Close the file with control + d i.e. `^D`

```bash
# Create the role tasks
cat > /home/thor/ansible/role/httpd/tasks/main.yml
```

```yaml
---
# tasks file for role/test

- name: install the latest version of HTTPD
  yum:
    name: httpd
    state: latest

- name: Start service httpd
  service:
    name: httpd
    state: started

- name: Template a file to /etc/file.conf
  template:
    src: templates/index.html.j2
    dest: /var/www/html/index.html
    owner: "{{ ansible_user }}"
    group: "{{ ansible_user }}"
    mode: '0755'
```

```bash
# Run the playbook
ansible-playbook -i inventory playbook.yml

# Check the deployment
curl -Lv http://stapp01
```

```
* Rebuilt URL to: http://stapp01/
*   Trying 172.16.238.10...
* TCP_NODELAY set
* Connected to stapp01 (172.16.238.10) port 80 (#0)
> GET / HTTP/1.1
> Host: stapp01
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Thu, 18 Jan 2024 03:11:11 GMT
< Server: Apache/2.4.37 (CentOS Stream)
< Last-Modified: Thu, 18 Jan 2024 03:10:20 GMT
< ETag: "2f-60f2fb422b726"
< Accept-Ranges: bytes
< Content-Length: 47
< Content-Type: text/html; charset=UTF-8
< 
This file was created using Ansible on stapp01
* Connection #0 to host stapp01 left intact
```

We are done.
