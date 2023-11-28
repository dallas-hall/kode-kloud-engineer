# Git Clone

## Task

> The Nautilus application development team has been working on a project repository `/opt/media.git`. This repo is cloned at `/usr/src/kodekloudrepos` on storage server in Stratos DC. They recently shared the following requirements with the DevOps team:
>
> * There are two branches in this repository, `master` and `feature`. One of the developers is working on the `feature` branch and their work is still in progress, however they want to merge one of the commits from the `feature` branch to the `master` branch, the message for the commit that needs to be merged into master is `Update info.txt`. Accomplish this task for them, also remember to push your changes eventually.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `git cherry-pick`
  * https://stackoverflow.com/questions/5304391/how-to-cherry-pick-from-1-branch-to-another

## Steps

```bash
# Connect to the storage server
ssh natasha@ststor01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*release*

# Switch to root
sudo -i

# Go to the correct path
cd /usr/src/kodekloudrepos/media

# Find what branch we are on, it was feature.
git status

# Get the commit hash that we can to cherry pick, which is the commit with the mesasge "Update info.txt"
git log
```

```
commit 80adc219c5b279e9730159be9216687262c19e47 (HEAD -> feature, origin/feature)
Author: Admin <admin@kodekloud.com>
Date:   Mon Nov 27 23:56:17 2023 +0000

    Update welcome.txt

commit d3a154d7b5273d841e7bd2fac9cb4243135045ee
Author: Admin <admin@kodekloud.com>
Date:   Mon Nov 27 23:56:17 2023 +0000

    Update info.txt

commit 66ea0025daea8b319d7661d0d6592919af63baca (origin/master, master)
Author: Admin <admin@kodekloud.com>
Date:   Mon Nov 27 23:56:17 2023 +0000

    Add welcome.txt

commit 18a2e75cb25caac35176a608cb266cd47e534f4c
Author: Admin <admin@kodekloud.com>
Date:   Mon Nov 27 23:56:17 2023 +0000

    initial commit
```

```bash
# Change to master
git checkout master

# Cherry pick the correct commit
git cherry-pick d3a154d7b5273d841e7bd2fac9cb4243135045ee
```

```
[master e6c8bfa] Update info.txt
 Date: Mon Nov 27 23:56:17 2023 +0000
 1 file changed, 1 insertion(+), 1 deletion(-)
```

```bash
# Push the changes
git push
```

We are done.
