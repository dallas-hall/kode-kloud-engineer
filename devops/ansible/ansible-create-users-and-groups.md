
# Ansible Create Users & Groups

## Task

> Several new developers and DevOps engineers just joined the xFusionCorp industries. They have been assigned the Nautilus project, and as per the onboarding process we need to create user accounts for new joinees on at least one of the app servers in Stratos DC. We also need to create groups and make new users members of those groups. We need to accomplish this task using Ansible. Below you can find more information about the task.
> * There is already an inventory file `~/playbooks/inventory` on jump host.
> * On jump host itself there is a list of users in `~/playbooks/data/users.yml` file and there are two groups — `admins` and `developers` —that have list of different users. Create a playbook `~/playbooks/add_users.yml` on jump host to perform the following tasks on app server 3 in Stratos DC.
> * Add all users given in the `users.yml` file on app server 3.
> * Also add `developers` and `admins` groups on the same server.
> * As per the list given in the `users.yml` file, make each user member of the respective group they are listed under.
> * Make sure home directory for all of the users under developers group is `/var/www` (not the default i.e `/var/www/{USER}`). Users under admins group should use the default home directory (i.e `/home/devid` for user `devid`).
> * Set password `8FmzjvFU6S` for all of the users under developers group and `GyQkFRVNr3` for of the users under admins group. Make sure to use the password given in the `~/playbooks/secrets/vault.txt` file as Ansible vault password to encrypt the original password strings. You can use `~/playbooks/secrets/vault.txt` file as a vault secret file while running the playbook (make necessary changes in `~/playbooks/ansible.cfg` file).
> * All users under admins group must be added as `sudo` users. To do so, simply make them member of the `wheel` group as well

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Ansible Vault password file environment variable.
  * https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-vault-password-file
* Generate the hashed password with a random salt.
  * https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-encrypted-passwords-for-the-user-module
* Encrypt the hashed and salted password with our Ansible Vault file.
  * https://docs.ansible.com/ansible/latest/user_guide/vault.html#use-encrypt-string-to-create-encrypted-variables-to-embed-in-yaml

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
# Change to working directory
cd ~/playbooks

# Inspect inventory
cat inventory

# Test inventory
ansible -i inventory all -m ping

# Update Ansible config file for Ansible Vault
cat >> ~/playbooks/ansible.cfg
```

```

vault_password_file = /home/thor/playbooks/secrets/vault.txt
```

The newline above is needed as the last command didn't have one. Close the file with control + d i.e. `^D`

```bash
# Create the hashed passwords
ansible -i inventory localhost -m debug -a "msg={{ '8FmzjvFU6S' | password_hash('sha512') }}" && ansible -i inventory localhost -m debug -a "msg={{ 'GyQkFRVNr3' | password_hash('sha512') }}"
```

```
ocalhost | SUCCESS => {
    "msg": "$6$qGSxL98fAEiVIHJQ$phGPofE1uKrOrngFW56M632dEwTC4BpmaqfcH.XChQY93bajIEA1jR5bCo.ta90XIoTaYdVMmhJhiuWcg6GxC1"
}
localhost | SUCCESS => {
    "msg": "$6$zidMi4auO1rom.4K$KwLGLNQHlIKzhqZ.Y85bMUseSfhanLDxldsjUkRpORtuOFDCZEzCwlc/71tVLM5V4HOwWKhg33BCwI1LDM3ex."
}
```

```bash
# Encrypt the passowrd hashes.
ansible-vault encrypt_string '$6$qGSxL98fAEiVIHJQ$phGPofE1uKrOrngFW56M632dEwTC4BpmaqfcH.XChQY93bajIEA1jR5bCo.ta90XIoTaYdVMmhJhiuWcg6GxC1' && ansible-vault encrypt_string '$6$zidMi4auO1rom.4K$KwLGLNQHlIKzhqZ.Y85bMUseSfhanLDxldsjUkRpORtuOFDCZEzCwlc/71tVLM5V4HOwWKhg33BCwI1LDM3ex.'
```

```
!vault |
          $ANSIBLE_VAULT;1.1;AES256
          39646164623362323033616264616233633435626565386335333763316138346539316235636538
          3935303264383164383962333132366161653361643234370a333761386161613964366566653839
          34303164326136666234373436633837646635383130323736366636623830373934656365316162
          3063336664393832650a306264666332356366653037653137323937653432336365626365643830
          34333237666363386639633338626362656331316663316139626235646435313161366535373732
          36356266323365313266636232376164303236326536303936393538613332663562306363396534
          35356165393232363734366538303262666531366233663633363239353232636338363739343531
          66643439323531653639313861343231303331343061373432646363326235613861643434613162
          39623430386232363830303937393962613632303239353761393466373033653939
Encryption successful
!vault |
          $ANSIBLE_VAULT;1.1;AES256
          38366662356432336635633266356365306639366531646133373863643333613664366461373066
          3736623634383337336337323037353733356562663165630a396535306265333765323434373736
          37666631333263656165616463613962373665363863363764643231633635613561316336323161
          3533386133323136300a376337636138383938366337303161333739323937313938363437363737
          37646664336531323133646233363961663535643538346462653961346562363733393135633865
          37633939653030353564393237323335326334323332636461663233393761643231636132376238
          63623335376331353637336663396636396665363234653032383637393862303562373161313364
          37306437666161643931306538613838343562386431383734366437313165666534656431343931
          37656334666131656663616661633939666162383164623934333338303836626661
```

```bash
# Create playbook
cat > add_users.yml
```

```yaml
---
- name: Add users and groups.
  hosts: stapp03
  become: yes
  gather_facts: yes

  tasks:
    # https://docs.ansible.com/ansible/latest/modules/group_module.html
    - name: Ensure the groups are present.
      group:
        name: "{{ item }}"
        state: present
      loop:
        - admins
        - developers

    # https://docs.ansible.com/ansible/latest/modules/user_module.html
    - name: Add developers.
      user:
        name: "{{ item }}"
        home: "/var/www/{{ item }}"
        groups: developers
        append: yes
        # https://stackoverflow.com/a/29910294
        password: "{{ user_password }}"
      loop:
        - tim
        - ray
        - jim
        - mark
      # https://stackoverflow.com/a/47567783
      vars:
        user_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          39646164623362323033616264616233633435626565386335333763316138346539316235636538
          3935303264383164383962333132366161653361643234370a333761386161613964366566653839
          34303164326136666234373436633837646635383130323736366636623830373934656365316162
          3063336664393832650a306264666332356366653037653137323937653432336365626365643830
          34333237666363386639633338626362656331316663316139626235646435313161366535373732
          36356266323365313266636232376164303236326536303936393538613332663562306363396534
          35356165393232363734366538303262666531366233663633363239353232636338363739343531
          66643439323531653639313861343231303331343061373432646363326235613861643434613162
          39623430386232363830303937393962613632303239353761393466373033653939
      
    - name: Add admins.
      user:
        name: "{{ item }}"
        groups: admins,wheel
        append: yes
        password: "{{ user_password }}"
      loop:
        - rob
        - david
        - joy
      vars:
        user_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          38366662356432336635633266356365306639366531646133373863643333613664366461373066
          3736623634383337336337323037353733356562663165630a396535306265333765323434373736
          37666631333263656165616463613962373665363863363764643231633635613561316336323161
          3533386133323136300a376337636138383938366337303161333739323937313938363437363737
          37646664336531323133646233363961663535643538346462653961346562363733393135633865
          37633939653030353564393237323335326334323332636461663233393761643231636132376238
          63623335376331353637336663396636396665363234653032383637393862303562373161313364
          37306437666161643931306538613838343562386431383734366437313165666534656431343931
          37656334666131656663616661633939666162383164623934333338303836626661
```

Close the file with control + d i.e. `^D`


```bash
# Run the playbook.
ansible-playbook -i inventory add_users.yml

# Check results
ssh banner@stapp03

# Check developer password
su - tim # Ok, password dCV3szSGNA worked.

# Check admin password
su - rob # Ok, password BruCStnMT5 worked.

# Check admin can sudo
sudo -i # Ok, changed to root.

# Check home directories
ls -Ahl --color /home # Ok, saw the expected admin users
ls -Ahl --color  /var/www # Ok, saw the expected developer users
```

We are done.
