# httpd Role

## Task

> One of the Nautilus DevOps team members is working on to develop a role for httpd installation and configuration. Work is almost completed, however there is a requirement to add a jinja2 template for index.html file. Additionally, the relevant task needs to be added inside the role. The inventory file `~/ansible/inventory` is already present on jump host that can be used. Complete the task as per details mentioned below:<br><br>a. Update `~/ansible/playbook.yml` playbook to run the httpd role on App Server 1.<br>b. Create a jinja2 template `index.html.j2` under `/home/thor/ansible/role/httpd/templates/` directory and add a line This file was created using Ansible on `<respective server>` (for example This file was created using Ansible on stapp01 in case of App Server 1). Also please make sure not to hard code the server name inside the template. Instead, use `inventory_hostname` variable to fetch the correct value.<br>c. Add a task inside `/home/thor/ansible/role/httpd/tasks/main.yml` to copy this template on App Server 1 under `/var/www/html/index.html`. Also make sure that `/var/www/html/index.html` file's permissions are `0644`.<br>d. The user/group owner of `/var/www/html/index.html` file must be respective sudo user of the server (for example `tony` in case of stapp01).

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
# Update playbook.
cat > playbook.yml
```
```
---
- hosts: stapp01
  become: yes
  become_user: root
  roles:
    - role/httpd
```

```bash
# Update template.
cat > /home/thor/ansible/role/httpd/templates/index.html.j2
```

```
This file was created using Ansible on {{ inventory_hostname }}
```
```bash
# Update the role.
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
    owner: tony
    group: tony
    mode: '0644'
```

```bash
# Run the playbook
ansible-playbook -i inventory playbook.yml
```

```
...
PLAY RECAP ****************************************************************************************************************************************************************
stapp01                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```bash
# Check the deployment
curl -v http://stapp01
```

```
* About to connect() to stapp01 port 80 (#0)
*   Trying 172.16.238.10...
* Connected to stapp01 (172.16.238.10) port 80 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: stapp01
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Mon, 24 Jul 2023 08:58:26 GMT
< Server: Apache/2.4.6 (CentOS)
< Last-Modified: Mon, 24 Jul 2023 08:58:17 GMT
< ETag: "2f-60137d0c9f070"
< Accept-Ranges: bytes
< Content-Length: 47
< Content-Type: text/html; charset=UTF-8
<
This file was created using Ansible on stapp01
* Connection #0 to host stapp01 left intact
```

We are done.
