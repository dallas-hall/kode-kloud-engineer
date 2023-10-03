# Grant Passwordless Sudo To User

## Task

> We have some users on all app servers in Stratos Datacenter. Some of them have been assigned some new roles and responsibilities, therefore their users need to be upgraded with sudo access so that they can perform admin level tasks.
>
> a. Provide `sudo` access to user `ravi` on all app servers.
>
> Make sure you have set up password-less sudo for the user.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Add user to a group
  * https://www.howtogeek.com/50787/add-a-user-to-a-group-or-second-group-on-linux/
* Passwordless sudo
  * https://www.cyberciti.biz/faq/linux-unix-running-sudo-command-without-a-password/
  * https://superuser.com/questions/869144/why-does-the-system-have-etc-sudoers-d-how-should-i-edit-it

## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Check if the user has sudo access via the group wheel
grep wheel /etc/group
```

```
wheel:x:10:ansible,tony
```

```bash
# Add user to group wheel for sudo access
usermod -aG wheel ravi

# Edit the sudoers file to grant passwordless sudo
visudo
```

You could also use the `/etc/sudoers.d/` way as well.

```
...
ravi     ALL=(ALL)   NOPASSWD:ALL
```

Close and save the file with `:x`

```bash
# Switch to the user
sudo -iu ravi

# Check passwordless sudo
sudo head -n 1 /etc/shadow
```

```
root:$6$jQ/8ISYU9MlFxz$XC0PrSuMHsYD9wnvh90UQza.fOo3hy6NqWxac2dqsWIoYRuuZ/.7HKLiDE.SiPbH7oGtInf6wVyzd6OYJ8Isd0:19422:0:99999:7:::
...
```

```bash
# Repeat for the other servers
ssh steve@stapp02
ssh banner@stapp03
```

We are done.
