# Ansible Conditionals

## Task

> The Nautilus DevOps team had a discussion about, how they can train different team members to use Ansible for different automation tasks. There are numerous ways to perform a particular task using Ansible, but we want to utilize each aspect that Ansible offers. The team wants to utilise Ansible's conditionals to perform the following task:
> * An inventory file is already placed under `/home/thor/ansible` directory on jump host, with all the Stratos DC app servers included.
> * Create a playbook `/home/thor/ansible/playbook.yml` and make sure to use Ansible's `when` conditionals statements to perform the below given tasks.
> * Copy `blog.txt` file present under `/usr/src/data` directory on jump host to App Server 1 under `/opt/data` directory. Its user and group owner must be user `tony` and its permissions must be `0744`.
> * Copy `story.txt` file present under `/usr/src/data` directory on jump host to App Server 2 under `/opt/data` directory. Its user and group owner must be user `steve` and its permissions must be `0744`.
> * Copy `media.txt` file present under `/usr/src/data` directory on jump host to App Server 3 under `/opt/data` directory. Its user and group owner must be user `banner` and its permissions must be `0744`.
>
> **NOTE:** You can use `ansible_nodename` variable from gathered facts with when condition. Additionally, please make sure you are running the play for all hosts i.e use - `hosts: all`.
>
> **Note:** Validation will try to run the playbook using command `ansible-playbook -i inventory playbook.yml`, so please make sure the playbook works this way without passing any extra arguments

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Copy module
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html
* Special variables.
  * https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#term-inventory_hostname
  * https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#term-ansible_user
* Ansible facts
  * https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html
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
- hosts: all
  become: yes
  become_user: root
  gather_facts: true
  tasks:
  - name: Copy file to stapp01
    ansible.builtin.copy:
      src: /usr/src/data/blog.txt
      dest: /opt/data
      owner: "{{ ansible_user }}"
      group: "{{ ansible_user }}"
      mode: '0744'
    when: ansible_nodename == 'stapp01.stratos.xfusioncorp.com'

  - name: Copy file to stapp02
    ansible.builtin.copy:
      src: /usr/src/data/story.txt
      dest: /opt/data
      owner: "{{ ansible_user }}"
      group: "{{ ansible_user }}"
      mode: '0744'
    when: ansible_nodename == 'stapp02.stratos.xfusioncorp.com'

  - name: Copy file to stapp03
    ansible.builtin.copy:
      src: /usr/src/data/media.txt
      dest: /opt/data
      owner: "{{ ansible_user }}"
      group: "{{ ansible_user }}"
      mode: '0744'
    when: ansible_nodename == 'stapp03.stratos.xfusioncorp.com'

```

Close the file with control + d i.e. `^D`


```bash
# Run the playbook
ansible-playbook -i inventory playbook.yml

# Check the deployment
ansible -i inventory stapp01 -m shell -a 'ls -Ahl /opt/data/blog.txt' & ansible -i inventory stapp02 -m shell -a 'ls -Ahl /opt/data/story.txt' & ansible -i inventory stapp03 -m shell -a 'ls -Ahl /opt/data/media.txt'
```

```
stapp03 | CHANGED | rc=0 >>
-rwxr--r-- 1 banner banner 22 Jan 18 03:29 /opt/data/media.txt
stapp01 | CHANGED | rc=0 >>
-rwxr--r-- 1 tony tony 35 Jan 18 03:29 /opt/data/blog.txt
stapp02 | CHANGED | rc=0 >>
-rwxr--r-- 1 steve steve 27 Jan 18 03:29 /opt/data/story.txt
```

We are done.
