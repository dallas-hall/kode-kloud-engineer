# Git Manage Remotes

## Task

> The xFusionCorp development team added updates to the project that is maintained under `/opt/official.git` repo and cloned under `/usr/src/kodekloudrepos/official`. Recently some changes were made on Git server that is hosted on Storage server in Stratos DC. The DevOps team added some new Git remotes, so we need to update remote on `/usr/src/kodekloudrepos/official` repository as per details mentioned below:
>
>* a. In `/usr/src/kodekloudrepos/official` repo add a new remote `dev_official` and point it to `/opt/xfusioncorp_official.git` repository.
* b. There is a file `/tmp/index.html` on same server; copy this file to the repo and `add`/`commit` to `master` branch.
* c. Finally `push` `master` branch to this new remote origin.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

None.

## Steps

```bash
# Connect to the Storage Server
ssh natasha@ststor01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Update remotes
cd /usr/src/kodekloudrepos/official
git remote update
```

```
Fetching origin
```

```bash
# Check remotes
git remote  -v
```

```
origin  /opt/official.git (fetch)
origin  /opt/official.git (push)
```

```bash
# Add new remote
git remote add dev_official /opt/xfusioncorp_official.git

# Check branch.
git status
```

```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

```bash
# Copy and commit file
cp /tmp/index.html .
git add index.html
git commit -m 'Added index.html'
```

```
1 file changed, 10 insertions(+)
 create mode 100644 index.html
```

```bash
# Push the changes to the desired repo and branch
git push dev_official master
```

```
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 36 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 585 bytes | 585.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
To /opt/xfusioncorp_official.git
 * [new branch]      master -> master
```

We are done.
