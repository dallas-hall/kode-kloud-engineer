# Git Merge Branches

## Task

> The Nautilus application development team has been working on a project repository `/opt/news.git`. This repo is cloned at `/usr/src/kodekloudrepos` on storage server in Stratos DC. They recently shared the following requirements with DevOps team:
>
> Create a new branch `xfusion` in `/usr/src/kodekloudrepos/news` repo from `master` and copy the `/tmp/index.html` file (present on storage server itself) into the repo. Further, `add`/`commit` this file in the new branch and `merge` back that branch into `master` branch. Finally, `push` the changes to the origin for both of the branches.

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
cd /usr/src/kodekloudrepos/news

# View watch branch we are on.
git status
```

```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

```bash
# Create a new branch.
git checkout -b xfusion
```

```
Switched to a new branch 'xfusion'
```

```bash
# Copy the file.
cp /tmp/index.html .

# Add and commit the file.
git add index.html
git commit -m 'Added index.html'
```

```
[xfusion 23676c6] Added index.html
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
```

```bash
# Switch to master.
git checkout master
```

```
Switched to branch 'master'
Your branch is up to date with 'origin/master'
```

```bash
# Merge the feature branch.
git merge xfusion
```

```
Updating cece96a..23676c6
Fast-forward
 index.html | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
```

```bash
# Push the master branch.
git push origin master
```

```
numerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 36 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 331 bytes | 331.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To /opt/news.git
   cece96a..23676c6  master -> master
```

```bash
# Switch to feature.
git checkout xfusion
```

```
Switched to branch 'xfusion'
```

```bash
# Push the feature branch.
git push origin xfusion
```

```
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To /opt/news.git
 * [new branch]      xfusion -> xfusion
```

We are done.
