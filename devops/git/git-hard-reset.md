# Git Reset Hard

## Task

> The Nautilus application development team was working on a git repository `/usr/src/kodekloudrepos/blog` present on Storage server in Stratos DC. This was just a test repository and one of the developers just pushed a couple of changes for testing, but now they want to clean this repository along with the commit history/work tree, so they want to point back the `HEAD` and the branch itself to a commit with message `add data.txt file`. Find below more details:
>
> * In `/usr/src/kodekloudrepos/blog` git repository, reset the git commit history so that there are only two commits in the commit history i.e initial commit and add data.txt file.
> * Also make sure to push your changes.



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
cat /etc/*release*

# Switch to root
sudo -i

# Go to the correct path
cd /usr/src/kodekloudrepos/blog

# View the current branch, it was master.
git status

# View the log.
git log
```


```
...
commit c2c0c04bee5d2f91dc204026f46324b9f8fdc72e
Author: Admin <admin@kodekloud.com>
Date:   Tue Nov 28 00:30:00 2023 +0000

    add data.txt file

commit 89db7b4aad48498d227959c88152d10d0f30dfdd
Author: Admin <admin@kodekloud.com>
Date:   Tue Nov 28 00:30:00 2023 +0000

    initial commit
```

```bash
# Reset back to the second commit
git reset --hard c2c0c04bee5d2f91dc204026f46324b9f8fdc72e
```

```
HEAD is now at c2c0c04 add data.txt file
```

```bash
# Push the changes with force
git push -f
```

```
otal 0 (delta 0), reused 0 (delta 0), pack-reused 0
To /opt/blog.git
 + 91f514d...c2c0c04 master -> master (forced update)
```

We are done.
