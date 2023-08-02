# Git Delete Branch

## Task

> Nautilus developers are actively working on one of the project repositories, `/usr/src/kodekloudrepos/ecommerce`. They were doing some testing and created few test branches, now they want to clean those test branches. Below are the requirements that have been shared with the DevOps team:<br>On Storage server in Stratos DC delete a branch named `xfusioncorp_ecommerce` from `usr/src/kodekloudrepos/ecommerce` git repo.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

None.

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

# View the branches
git branch -v
```

```
 master                4b7d1df initial commit
* xfusioncorp_ecommerce 4b7d1df initial commit
```

```bash
# Switch branch
git checkout master
```

```
git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

```bash
# Delete the branch
git branch -D xfusioncorp_ecommerce
```

```
Deleted branch xfusioncorp_ecommerce (was 4b7d1df).
```

We are done.
