# Git Rebase

## Task

> The Nautilus application development team has been working on a project repository `/opt/news.git`. This repo is cloned at `/usr/src/kodekloudrepos` on storage server in Stratos DC. They recently shared the following requirements with DevOps team:
>
> * One of the developers is working on `feature` branch and their work is still in progress, however there are some changes which have been pushed into the `master` branch, the developer now wants to rebase the `feature` branch with the `master` branch without loosing any data from the `feature` branch, also they don't want to add any merge commit by simply merging the `master` branch into the `feature` branch. 
> 
> Accomplish this task as per requirements mentioned.



Also remember to push your changes once done.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `git rebase`
  * https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase
  * https://www.atlassian.com/git/tutorials/merging-vs-rebasing
  * https://git-scm.com/docs/git-rebase

## Steps

```bash
# Connect to the storage server
ssh natasha@ststor01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root.
sudo -i

# Go to the repo.
cd /usr/src/kodekloudrepos/news

# View watch branch we are on.
git status
```

```
On branch feature
nothing to commit, working tree clean
```

```bash
# Rebase master into the feature branch.
git rebase master
```

```
Successfully rebased and updated refs/heads/feature.
```

```bash
# View the log
git log
```

```
commit e5f2b96b7a207e62b78b25930c85eeb30258d279 (HEAD -> feature)
Author: Admin <admin@kodekloud.com>
Date:   Mon Jan 15 02:27:27 2024 +0000

    Add new feature

commit 1bc232e6fdba4b9549017ff43bd8fb3f2c9f7489 (origin/master, master)
Author: Admin <admin@kodekloud.com>
Date:   Mon Jan 15 02:27:27 2024 +0000

    Update info.txt

commit f84cb4407e7706bbf426dea3bcbd982b4d3d63e4
Author: Admin <admin@kodekloud.com>
Date:   Mon Jan 15 02:27:27 2024 +0000

    initial commit
```

```bash
# Push the feature branch.
git push origin feature
```

```
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 36 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 315 bytes | 315.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To /opt/news.git
   d2f91df..827ca05  feature -> feature
```

We are done.
