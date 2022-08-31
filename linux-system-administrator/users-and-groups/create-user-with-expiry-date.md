# Create A User With Expiry Date

## Task

> A developer ammar has been assigned Nautilus project temporarily as a backup resource. As a temporary resource for this project, we need a temporary user for ammar. It’s a good idea to create a user with a set expiration date so that the user won't be able to access servers beyond that point.<br><br>Therefore, create a user named `ammar` on the App Server 3. Set expiry date to 2021-04-15 in Stratos Datacenter. Make sure the user is created as per standard and is in lowercase.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create the user with an expiry date - https://unix.stackexchange.com/questions/80968/how-can-i-create-automatically-expiring-user-accounts
* Check expiry time - https://www.cyberciti.biz/faq/linux-howto-check-user-password-expiration-date-and-time/

## Steps

```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Create the user with an expiry date
useradd -e 2021-04-15 ammar

# Check expiry time
chage -l ammar
```

```
Last password change                                    : Aug 31, 2022
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Apr 15, 2021
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```
