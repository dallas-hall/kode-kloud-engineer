# Git Clean

## Task

> The Nautilus application development team was working on a git repository `/usr/src/kodekloudrepos/ecommerce` present on Storage server in Stratos DC. One of the developers mistakenly created a couple of files under this repository, but now they want to clean this repository without adding/pushing any new files. Find below more details:
>
> * Clean the `/usr/src/kodekloudrepos/ecommerce` git repository without adding/pushing any new files, make sure `git status` is clean.


## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Git clean
  * https://git-scm.com/docs/git-clean

## Steps

```bash
# Connect to the storage server
ssh natasha@ststor01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*release*

# Switch to root
sudo -i

# Go to the correct path
cd /usr/src/kodekloudrepos/ecommerce

# Remove the junk
git clean -f
```

We are done.
