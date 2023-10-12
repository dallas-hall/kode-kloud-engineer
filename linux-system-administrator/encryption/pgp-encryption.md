# PGP Encryption With GPG

## Task

> We have confidential data that needs to be transferred to a remote location, so we need to encrypt that data.We also need to decrypt data we received from a remote location in order to understand its content.
>
> On storage server in Stratos Datacenter we have private and public keys stored at `/home/*_key.asc.` Use these keys to perform the following actions.
> * Encrypt `/home/encrypt_me.txt` to `/home/encrypted_me.asc`.
> * Decrypt `/home/decrypt_me.asc` to `/home/decrypted_me.txt`. (Passphrase for decryption and encryption is `kodekloud`).
>* The user ID you can use is `kodekloud@kodekloud.com`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* What is PGP encryption.
  * https://www.varonis.com/blog/pgp-encryption
* How to use PGP encryption.
  * https://www.howtogeek.com/427982/how-to-encrypt-and-decrypt-files-with-gpg-on-linux
  * https://www.gnupg.org/gph/en/manual.html#INTRO

## Steps

```bash
# Connect to the first app server.
ssh natasha@ststor01

# Switch to root.
sudo -i

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# View the keys
cd /home
ls -Ahl --color
```

```
drwx------ 1 ansible ansible 4.0K Mar  6  2023 ansible
-rw-r--r-- 1 root    root     155 Oct 12 08:48 decrypt_me.asc
-rw-r--r-- 1 root    root      99 Oct 12 08:51 encrypt_me.txt
drwx------ 1 natasha natasha 4.0K Apr 11  2023 natasha
-rw-r--r-- 1 root    root    3.6K Oct 12 08:51 private_key.asc
-rw-r--r-- 1 root    root    1.7K Oct 12 08:51 public_key.asc
```

```bash
# Import the public key
gpg --import public_key.asc
```

```
gpg: directory '/root/.gnupg' created
gpg: keybox '/root/.gnupg/pubring.kbx' created
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key 8F17F26ECCE3AF51: public key "kodekloud <kodekloud@kodekloud.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

```bash
# View our key
gpg --list-keys
```

```
/root/.gnupg/pubring.kbx
------------------------
pub   rsa2048 2020-01-19 [SC]
      FEA85011C456B5E9AE5A516F8F17F26ECCE3AF51
uid           [ unknown] kodekloud <kodekloud@kodekloud.com>
sub   rsa2048 2020-01-19 [E]
```

```bash
# Encrypt file with a password using symmetric encryption.
gpg --output /home/encrypted_me.asc --encrypt --recipient kodekloud@kodekloud.com --symmetric /home/encrypt_me.txt
```

Enter `kodekloud` as the password at both prompts.

```
gpg: DD6B8506865C070D: There is no assurance this key belongs to the named user

sub  rsa2048/DD6B8506865C070D 2020-01-19 kodekloud <kodekloud@kodekloud.com>
 Primary key fingerprint: FEA8 5011 C456 B5E9 AE5A  516F 8F17 F26E CCE3 AF51
      Subkey fingerprint: 7B4B 5CFC 5E4F B4B6 EEC0  83E5 DD6B 8506 865C 070D

It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.

Use this key anyway? (y/N) y
```

```bash
# Decrypt file with password
gpg --decrypt --output /home/decrypted_me.txt /home/decrypt_me.asc
```

Enter `kodekloud` as the password.

```
pg: AES encrypted data
gpg: encrypted with 1 passphrase
```

```bash
# View files
ls -Ahl --color /home
```

```
drwx------ 1 ansible ansible 4.0K Mar  6  2023 ansible
-rw-r--r-- 1 root    root     155 Oct 12 08:48 decrypt_me.asc
-rw-r--r-- 1 root    root      80 Oct 12 08:58 decrypted_me.txt
-rw-r--r-- 1 root    root      99 Oct 12 08:51 encrypt_me.txt
-rw-r--r-- 1 root    root     483 Oct 12 08:57 encrypted_me.asc
drwx------ 1 natasha natasha 4.0K Apr 11  2023 natasha
-rw-r--r-- 1 root    root    3.6K Oct 12 08:51 private_key.asc
-rw-r--r-- 1 root    root    1.7K Oct 12 08:51 public_key.asc
```

We are done.
