# Copy Files With Find

## Task

> There was some users data copied on Nautilus App Server 1 at `/home/usersdata` location by the Nautilus production support team in Stratos DC. Later they found that they mistakenly mixed up different user data there. Now they want to filter out some user data and copy it to another location. Find the details below:<br><br>On App Server 1 find all files (not directories) owned by user james inside `/home/usersdata` directory and copy them all while keeping the folder structure (preserve the directories path) to `/official` directory.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Find all files owned by a user - https://www.cyberciti.biz/faq/how-do-i-find-all-the-files-owned-by-a-particular-user-or-group/
* Copy files with `find` keeping directory structure - https://unix.stackexchange.com/questions/83593/copy-specific-file-type-keeping-the-folder-structure

## Steps

```bash
# Connect to the first app server
ssh tony@stapp01

# Get root
sudo -i

# Check Linux version
cat /etc/*release*
```

```
CentOS Linux release 7.6.1810 (Core)
Derived from Red Hat Enterprise Linux 7.6 (Source)
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

CentOS Linux release 7.6.1810 (Core)
CentOS Linux release 7.6
```

```bash
# Find all files owned by the user
find /home/usersdata -type f -user james

# Go to the correct directory, needed for copying properly
cd /home/usersdata

# Copy all the files owned by the user and preserve the folder structure and permissions
find . -type f -user james -exec cp -ar --parents '{}' /official ';'

# Check the copy
ls -Ahl /official
```

```
total 180K
-rw-r--r--  1 james root  420 Aug 12 05:45 index.php
-rw-r--r--  1 james root 6.8K Aug 12 05:45 wp-activate.php
drwxr-xr-x  7 root  root 4.0K Aug 12 05:59 wp-admin
-rw-r--r--  1 james root  369 Aug 12 05:45 wp-blog-header.php
-rw-r--r--  1 james root 2.3K Aug 12 05:45 wp-comments-post.php
-rw-r--r--  1 james root 2.9K Aug 12 05:45 wp-config-sample.php
drwxr-xr-x  4 root  root 4.0K Aug 12 05:45 wp-content
-rw-r--r--  1 james root 3.9K Aug 12 05:45 wp-cron.php
drwxr-xr-x 16 root  root  12K Aug 12 06:00 wp-includes
-rw-r--r--  1 james root 2.5K Aug 12 05:45 wp-links-opml.php
-rw-r--r--  1 james root 3.3K Aug 12 05:45 wp-load.php
-rw-r--r--  1 james root  47K Aug 12 05:45 wp-login.php
-rw-r--r--  1 james root 8.3K Aug 12 05:45 wp-mail.php
-rw-r--r--  1 james root  19K Aug 12 05:45 wp-settings.php
-rw-r--r--  1 james root  31K Aug 12 05:45 wp-signup.php
-rw-r--r--  1 james root 4.7K Aug 12 05:45 wp-trackback.php
-rw-r--r--  1 james root 3.1K Aug 12 05:45 xmlrpc.php
```
