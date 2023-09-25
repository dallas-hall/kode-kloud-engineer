# Git Create Branch

## Task

> Nautilus developers are actively working on one of the project repositories, `/usr/src/kodekloudrepos/media`. Recently, they decided to implement some new features in the application, and they want to maintain those new changes in a separate branch. Below are the requirements that have been shared with the DevOps team:
>
> * On Storage server in Stratos DC create a new branch `xfusioncorp_media` from `master` branch in `/usr/src/kodekloudrepos/media` git repo.

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
cat /etc/*rel*

# Switch to root.
sudo -i

# Go to the repo.
cd /usr/src/kodekloudrepos/media

# View watch branch we are on.
git status
```

```
On branch kodekloud_media
nothing to commit, working tree clean
```

```bash
# Switch to master.
git checkout master
```

```
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

```bash
# Create a new branch.
git checkout -b xfusioncorp_media
```

```
Switched to a new branch 'xfusioncorp_media'
```

We are done.
