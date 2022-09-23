# PGP Encryption With GPG

## Task

> We have confidential data that needs to be transferred to a remote location, so we need to encrypt that data.We also need to decrypt data we received from a remote location in order to understand its content.<br><br>On storage server in Stratos Datacenter we have private and public keys stored `/home/*_key.asc`. Use those keys to perform the following actions.<br>Encrypt `/home/encrypt_me.txt` to `/home/encrypted_me.asc`.<br>Decrypt `/home/decrypt_me.asc` to `/home/decrypted_me.txt`. (Passphrase for decryption and encryption is `kodekloud`).

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* What is PGP encryption - https://www.varonis.com/blog/pgp-encryption
* How to use PGP encryption - https://www.howtogeek.com/427982/how-to-encrypt-and-decrypt-files-with-gpg-on-linux/ & https://www.gnupg.org/gph/en/manual.html#INTRO

## Steps

```bash
# Connect to the first app server.
ssh natasha@ststor01

# Switch to root.
sudo -i

# Import the public key
cd /home
gpg --import public_key.asc
```

```
gpg: directory `/root/.gnupg' created
gpg: new configuration file `/root/.gnupg/gpg.conf' created
gpg: WARNING: options in `/root/.gnupg/gpg.conf' are not yet active during this run
gpg: keyring `/root/.gnupg/secring.gpg' created
gpg: keyring `/root/.gnupg/pubring.gpg' created
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key CCE3AF51: public key "kodekloud <kodekloud@kodekloud.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1
```

```bash
# View our key
gpg --list-keys
```

```
/root/.gnupg/pubring.gpg
------------------------
pub   2048R/CCE3AF51 2020-01-19
uid                  kodekloud <kodekloud@kodekloud.com>
sub   2048R/865C070D 2020-01-19
```

```bash
# Encrypt file with a password (i.e. symmetric)
gpg --output /home/encrypted_me.asc --encrypt --recipient kodekloud@kodekloud.com --symmetric /home/encrypt_me.txt
```

Enter `kodekloud` as the password at both prompts.

```
gpg: 865C070D: There is no assurance this key belongs to the named user

pub  2048R/865C070D 2020-01-19 kodekloud <kodekloud@kodekloud.com>
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
gpg: AES encrypted data
gpg: encrypted with 1 passphrase
```

```bash
# View files
ls -Ahl --color /home
```

```
drwx------ 1 ansible ansible 4.0K Oct 15  2019 ansible
-rw-r--r-- 1 root    root      80 Sep 23 05:53 decrypted_me.txt
-rw-r--r-- 1 root    root     155 Sep 23 05:32 decrypt_me.asc
-rw-r--r-- 1 root    root     483 Sep 23 05:52 encrypted_me.asc
-rw-r--r-- 1 root    root      99 Sep 23 05:43 encrypt_me.txt
drwx------ 1 natasha natasha 4.0K Jan 12  2020 natasha
-rw-r--r-- 1 root    root    3.6K Sep 23 05:43 private_key.asc
-rw-r--r-- 1 root    root    1.7K Sep 23 05:43 public_key.asc
```
