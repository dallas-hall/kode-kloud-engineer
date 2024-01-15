# Git Hooks

## Task

> The Nautilus application development team was working on a git repository `/opt/media.git` which is cloned under `/usr/src/kodekloudrepos` directory present on Storage server in Stratos DC. The team want to setup a hook on this repository, please find below more details:
> * Merge the `feature` branch into the `master` branch, but before pushing your changes complete below point.
> * Create a `post-update` hook in this git repository so that whenever any changes are pushed to the `master` branch, it creates a release tag with name `release-2023-06-15`, where 2023-06-15 is supposed to be the current date. For example if today is 20th June, 2023 then the release tag must be release-2023-06-20. Make sure you test the hook at least once and create a release tag for today's release.
> * Finally remember to push your changes.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Git hooks
  * https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks
  * https://git-scm.com/docs/githooks#post-update
  * https://www.atlassian.com/git/tutorials/git-hooks
* Git latest commit hash.
  * https://stackoverflow.com/questions/949314/how-do-i-get-the-hash-for-the-current-commit-in-git
* Linux `date`
  * https://www.geeksforgeeks.org/date-command-linux-examples/
* Git tags.
  * https://www.atlassian.com/git/tutorials/inspecting-a-repository/git-tag


## Steps

```bash
# Connect to the storage server
ssh natasha@ststor01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Create the post-update hook file on the server side repo.
# This file was taken from /opt/media.git/hooks/post-update.sample
cat > /opt/media.git/hooks/post-update
```

The contents should be:

```
#!/bin/sh
CURRENT_DATE=$(date "+%Y-%m-%d")
COMMIT_HASH=$(git rev-parse HEAD)
COMMIT_TAG="release-${CURRENT_DATE}"
exec git tag "${COMMIT_TAG}" "${COMMIT_HASH}" -m "Added new release tag."
```

```bash
# Make the hook file executable.
chmod 755 /opt/media.git/hooks/post-update

# Go to the cloned repo path.
cd /usr/src/kodekloudrepos/media

# View current branch, it was feature.
git status

# SWitch to master and merge feature
git checkout master
git merge feature
```

```
Updating b362834..ae05a36
Fast-forward
 feature.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
```

```bash
# Push the changes
git push origin master
```

```
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To /opt/media.git
   b362834..ae05a36  master -> master
```

```bash
# View the remote changes
git remote update
```

```
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (1/1), 151 bytes | 151.00 KiB/s, done.
From /opt/media
 * [new tag]         release-2024-01-15 -> release-2024-01-15
```

```bash
# View the new tag.
git log
```

```
commit ae05a36d4202f99570d407494ba922d385f97401 (HEAD -> master, tag: release-2024-01-15, origin/master, origin/feature, feature)
Author: Admin <admin@kodekloud.com>
Date:   Mon Jan 15 06:19:50 2024 +0000

    Add feature

commit b3628343ceeb1908ce087770245d48478301b3be
Author: Admin <admin@kodekloud.com>
Date:   Mon Jan 15 06:19:50 2024 +0000

    initial commit
```

We are done.