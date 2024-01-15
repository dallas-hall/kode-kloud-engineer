# Resolve Merge Conflicts

## Task

> Sarah and Max were working on writting some stories which they have pushed to the repository. Max has recently added some new changes and is trying to push them to the repository but he is facing some issues. Below you can find more details:
> * SSH into storage server using user `max` and password `Max_pass123`. Under `/home/max` you will find the `story-blog` repository. Try to push the changes to the `origin` repo and fix the issues. The `story-index.txt` must have titles for all 4 stories. Additionally, there is a typo in The Lion and the Mooose line where Mooose should be Mouse.
> * Click on the Gitea UI button on the top bar. You should be able to access the Gitea page. You can login to Gitea server from UI using username `sarah` and password `Sarah_pass123` or username `max` and password `Max_pass123`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

None.

## Steps

```bash
# Connect to the storage server
ssh max@ststor01

# Check current Linux version, it was Alpine Linux 3.4.6
cat /etc/*rel*

# Go to the repo.
cd story-blog

# View watch branch we are on.
git status
```

```
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
nothing to commit, working directory clean
```

```bash
# Get latest code
git remote update
```

```
Fetching origin
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From http://git.stratos.xfusioncorp.com/sarah/story-blog
   a4f509a..5be73c5  master     -> origin/master
```

```bash
# Get latest code.
git pull origin master
```

```
From http://git.stratos.xfusioncorp.com/sarah/story-blog
 * branch            master     -> FETCH_HEAD
Auto-merging story-index.txt
CONFLICT (add/add): Merge conflict in story-index.txt
Automatic merge failed; fix conflicts and then commit the result.
```

```bash
# Fix the merge conflicts.
vi story-index.txt
```

The contents should be:

```
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
```

```bash
# Commit and push the changes
git add story-index.txt
git commit -m 'Fixed typo and merge conflicts.'
git push origin master
```

```
Username for 'http://git.stratos.xfusioncorp.com': max
Password for 'http://max@git.stratos.xfusioncorp.com': 
Counting objects: 7, done.
Delta compression using up to 36 threads.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (7/7), 1.18 KiB | 0 bytes/s, done.
Total 7 (delta 1), reused 0 (delta 0)
remote: . Processing 1 references
remote: Processed 1 references in total
To http://git.stratos.xfusioncorp.com/sarah/story-blog.git
   5be73c5..f59d311  master -> master
```

Click the Gitea UI button, login, and check the changes. Everything was good. We are done.
