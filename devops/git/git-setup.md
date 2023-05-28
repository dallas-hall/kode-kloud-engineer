# Git Setup

## Task

> Some new developers have joined xFusionCorp Industries and have been assigned Nautilus project. They are going to start development on a new application, and some pre-requisites have been shared with the DevOps team to proceed with. Please note that all tasks need to be performed on storage server in Stratos DC.<br><br>a. Install `git`, set up any values for `user.email` and `user.name` globally and create a bare repository `/opt/news.git`.<br>b. There is an update hook (to block direct pushes to master branch) under `/tmp` on storage server itself; use the same to block direct pushes to master branch in `/opt/news.git` repo.<br>c. Clone `/opt/news.git` repo in `/usr/src/kodekloudrepos/news `directory.<br>d. Create a new branch `xfusioncorp_news` in repo that you cloned in `/usr/src/kodekloudrepo`s.<br>e. There is a `readme.md` file in `/tmp` on storage server itself; copy that to repo, add/commit in the new branch you created, and finally push your branch to `origin`.<br>f. Also create `master` branch from your branch and remember you should not be able to push to `master` as per hook you have set up.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create a bare repo.
  * https://git-scm.com/docs/git-init#Documentation/git-init.txt---bare
  * https://stackoverflow.com/a/7632497
* Create hook to block pushes to master .
  * https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks

## Steps

```bash
# Connect to the storage server
ssh natasha@ststor01

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo su

# Install git
yum install -y git
```

```
...
Installed:
  git.x86_64 0:1.8.3.1-23.el7_8
...
```

```bash
# Set up default user information.
git config --global user.name "root"
git config --global user.email "root@localhost"

# Create repo path
mkdir -p /opt/news.git
cd /opt/news.git
git init --bare
```

```
Initialized empty Git repository in /opt/news.git/.git/
```

```bash
# Create hook to block pushes to master
cp /tmp/update /opt/news.git/hooks/update

# Clone the repo to a new location
git clone . /usr/src/kodekloudrepos/news/
```

```
Cloning into '/usr/src/kodekloudrepos/news'...
warning: You appear to have cloned an empty repository.
done.
```

```bash
# Create new branch
cd /usr/src/kodekloudrepos/news
git checkout -b xfusioncorp_news
```

```
Switched to a new branch 'xfusioncorp_news'
```

```bash
# Create the initial commit
cp /tmp/readme.md .
git add readme.md
git commit -m 'Initial commit.'
```

```
[xfusioncorp_news (root-commit) 8b30108] Initial commit.
 1 file changed, 1 insertion(+)
 create mode 100644 readme.md
```

```bash
# Push the initial commit
git push origin xfusioncorp_news
```

```
Counting objects: 3, done.
Writing objects: 100% (3/3), 238 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To /opt/news.git/.
 * [new branch]      xfusioncorp_news -> xfusioncorp_news
```

Pushed and created a new branch like we expected.

```bash
# Create master branch
git checkout -b master
```

```
Switched to a new branch 'master'
```


```bash
# Check the pushing to master is blocked by the git hook
git push origin master
```

```
Total 0 (delta 0), reused 0 (delta 0)
remote: Manual pushing to this repo's master branch is restricted
remote: error: hook declined to update refs/heads/master
To /opt/news.git/.
 ! [remote rejected] master -> master (hook declined)
error: failed to push some refs to '/opt/news.git/.'
```

It was blocked like we expected.
