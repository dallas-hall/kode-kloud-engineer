# Ansible Inventory Update

## Task

> The Nautilus DevOps team has started testing their Ansible playbooks on different servers within the stack. They have placed some playbooks under `/home/thor/playbook/` directory on jump host which they want to test. Some of these playbooks have already been tested on different servers, but now they want to test them on app server 3 in Stratos DC. However, they first need to create an inventory file so that Ansible can connect to the respective app. Below are some requirements:<br><br>a. Create an ini type Ansible inventory file` /home/thor/playbook/inventory` on jump host.<br>b. Add App Server 3 in this inventory along with required variables that are needed to make it work.<br>c. The inventory hostname of the host should be the server name as per the wiki, for example stapp01 for app server 1 in Stratos DC.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Inventory - https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html

## Steps

Follow [Passwordless ssh setup](../../linux-system-administrator/networking/passwordless-ssh-access.md) for every user.


```bash
# Create inventory
cat > ~playbook/inventory
```

```yaml
# YAML format
all:
  hosts:
    stapp03:
      ansible_user: banner

    ansible_connection: ssh
```

```
# INI format
stapp03 ansible_host=172.16.238.12 ansible_connection=ssh ansible_user=banner
```

Close the file with control + d i.e. `^D`

```bash
# Test inventory
ansible -i ~/playbook/inventory all -m ping
```

```
stapp03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```


```bash
# Run playbook
ansible-playbook -i ~/playbook/inventory ~/playbook/playbook.yml
```

Looks good, we are done.
