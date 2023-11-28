# Git Stash

## Task

> The Nautilus application development team was working on a git repository `/usr/src/kodekloudrepos/ecommerce` present on Storage server in Stratos DC. One of the developers stashed some in-progress changes in this repository, but now they want to restore some of the stashed changes. Find below more details to accomplish this task:
> * Look for the stashed changes under `/usr/src/kodekloudrepos/ecommerce` git repository, and restore the stash with `stash@{1}` identifier.
> * Commit and push your changes to the `origin` remote.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Git stash
  * https://www.atlassian.com/git/tutorials/saving-changes/git-stash
  * https://git-scm.com/docs/git-stash

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

# List the stashes
git stash list
```

```
stash@{0}: WIP on master: b31a3bc initial commit
stash@{1}: WIP on master: b31a3bc initial commit
```

```bash
# Grab the correct stash
git stash pop stash@{1}
```

```
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   welcome.txt

Dropped stash@{1} (570844ae24c0ebd16421c3c903fbb9239a13a78b)
```

```bash
# Commit and push the changes
git add welcome.txt 
git commit -m 'Popped stash stash@{1}'
git push
```

We are done.
