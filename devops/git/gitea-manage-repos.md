# Gitea Manage Repos

## Task

> A new developer just joined the Nautilus development team and has been assigned a new project for which he needs to create a new repository under his account on Gitea server. Additionally, there is some existing data that need to be added to the repo. Below you can find more details about the task:<br><br>Click on the Gitea UI button on the top bar. You should be able to access the Gitea UI. Login to Gitea server using username `max` and password `Max_pass123`.<br>a. Create a new git repository `story_official` under `max` user.<br>b. SSH into storage server using user `max` and password `Max_pass123` and clone this newly created repository under user max home directory i.e `/home/max`.<br>c. Copy all files from location `/usr/finance` to the repository and commit/push your changes to the master branch. The commit message must be `"add stories"` (must be done in single commit).<br>d. Create a new branch `max_demo` from master.<br>e. Copy a file `story-index-max.txt` from location `/tmp/stories/` to the repository. This file has a typo, which you can fix by changing the word Mooose to Mouse. Commit and push the changes to the newly created branch. Commit message must be `"typo fixed for Mooose"` (must be done in single commit).


## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

N/A

## Steps

* Log into the UI
* Create the repository.

```bash
# Connect to the storage server
ssh max@ststor01

# Clone the repo
git clone http://git.stratos.xfusioncorp.com/max/story_official.git

# Copy the files
cp /usr/finance/* ~/story_official

# Commit the files
cd story_official
git add .
git commit -m "add stories"
```

```
[master (root-commit) f7475fb] add stories
 Committer: Linux User <max@ststor01.stratos.xfusioncorp.com>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 2 files changed, 42 insertions(+)
 create mode 100644 frogs-and-ox.txt
 create mode 100644 lion-and-mouse.txt
```

```bash
# Push the changes
git push origin master
```

```
Username for 'http://git.stratos.xfusioncorp.com': max
Password for 'http://max@git.stratos.xfusioncorp.com':
Counting objects: 4, done.
Delta compression using up to 36 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 1.19 KiB | 0 bytes/s, done.
Total 4 (delta 0), reused 0 (delta 0)
remote: . Processing 1 references
remote: Processed 1 references in total
To http://git.stratos.xfusioncorp.com/max/story_official.git
 * [new branch]      master -> master
```

```bash
# Create the new branch
git checkout -b max_demo
```

```
Switched to a new branch 'max_demo'
```

```bash
# Copy the files
cp /tmp/stories/story-index-max.txt ~/story_official/

# Fix typo
sed -i -r 's/Mooose/Mouse/g' story-index-max.txt

# Commit changes
git add story-index-max.txt
git commit -m "typo fixed for Mooose"
```

```
[max_demo e09773b] typo fixed for Mooose
 Committer: Linux User <max@ststor01.stratos.xfusioncorp.com>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 1 file changed, 4 insertions(+)
 create mode 100644 story-index-max.txt
```

```bash
# Push the changes
git push origin max_demo
```

```
Username for 'http://git.stratos.xfusioncorp.com': max
Password for 'http://max@git.stratos.xfusioncorp.com':
Counting objects: 3, done.
Delta compression using up to 36 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 414 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote:
remote: Create a new pull request for 'max_demo':
remote:   http://git.stratos.xfusioncorp.com/max/story_official/compare/master...max_demo
remote:
remote: . Processing 1 references
remote: Processed 1 references in total
To http://git.stratos.xfusioncorp.com/max/story_official.git
 * [new branch]      max_demo -> max_demo
```

We are done.
