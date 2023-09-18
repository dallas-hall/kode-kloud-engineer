# Ansible Unarchive Module

## Task

> One of the DevOps team members has created a zip archive on jump host in Stratos DC that needs to be extracted and copied over to all app servers in Stratos DC itself. Because this is a routine task, the Nautilus DevOps team has suggested automating it. We can use Ansible since we have been using it for other automation tasks. Below you can find more details about the task:
>
> We have an inventory file under `/home/thor/ansible` directory on jump host, which should have all the app servers added already.
>
> There is a zip archive `/usr/src/sysops/xfusion.zip` on jump host.
>
> 1. Create a `playbook.yml` under `/home/thor/ansible/` directory on jump host itself to perform the below given tasks.
> 2. Unzip `/usr/src/sysops/xfusion.zip` archive in `/opt/sysops/` location on all app servers.
> 3. Make sure the extracted data must has the respective `sudo` user as their user and group owner, i.e `tony` for app server 1, `steve` for app server 2, `banner` for app server 3.
> 4. The extracted data permissions must be `0744`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Unarchive.
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/unarchive_module.html
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
  - name: Extract an archive.
    unarchive:
      src: /usr/src/sysops/xfusion.zip
      dest: /opt/sysops/
      owner: "{{ ansible_user }}"
      group: "{{ ansible_user }}"
      mode: '0744'
```

Close the file with control + d i.e. `^D`

```bash
# Run the playbook
ansible-playbook -i inventory playbook.yml
```

```
...
PLAY RECAP *************************************************************************************************************************************
stapp01                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp02                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp03                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```bash
# Check the playbook work
ansible -i inventory all -m shell -a 'ls -Ahl /opt/sysops'
```

```
stapp01 | CHANGED | rc=0 >>
total 4.0K
drwxr--r-- 2 tony tony 4.0K Sep 18 05:25 unarchive
stapp02 | CHANGED | rc=0 >>
total 4.0K
drwxr--r-- 2 steve steve 4.0K Sep 18 05:25 unarchive
stapp03 | CHANGED | rc=0 >>
total 4.0K
drwxr--r-- 2 banner banner 4.0K Sep 18 05:25 unarchive
```

Looks good, we are done.
