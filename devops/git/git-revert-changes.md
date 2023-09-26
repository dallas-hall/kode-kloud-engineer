# Git Revert Changes

## Task

> The Nautilus application development team was working on a git repository `/usr/src/kodekloudrepos/demo` present on Storage server in Stratos DC. However, they reported an issue with the recent commits being pushed to this repo. They have asked the DevOps team to revert repo `HEAD` to last commit. Below are more details about the task:
>
> * In `/usr/src/kodekloudrepos/demo` git repository, revert the latest commit ( `HEAD` ) to the previous commit (JFYI the previous commit hash should be with initial commit message ).
> * Use revert demo message (please use all small letters for commit message) for the new revert commit.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Git revert.
  * https://stackoverflow.com/questions/19032296/how-to-use-git-revert

## Steps

```bash
# Connect to the storage server
ssh natasha@ststor01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root.
sudo -i

# Go to the repo.
cd /usr/src/kodekloudrepos/demo

# View the log.
git log
```

```
commit 0e442b89ca8efe1af6273dda012acc83991361e9 (HEAD -> master, origin/master)
Author: Admin <admin@kodekloud.com>
Date:   Tue Sep 26 09:01:11 2023 +0000

    add data.txt file

commit 8e3fef7312b44bc9b29d9b7ee4f62a3cb4649ff9
Author: Admin <admin@kodekloud.com>
Date:   Tue Sep 26 09:01:11 2023 +0000

    initial commit
```

```bash
# Use git revert.
git revert
```

Add `revert demo` and press `:x` to save and close the commit message.

# View the log.
git log
```

```
Author: Admin <admin@kodekloud.com>
Date:   Tue Sep 26 09:06:37 2023 +0000

    revert demo

commit 0e442b89ca8efe1af6273dda012acc83991361e9 (origin/master)
Author: Admin <admin@kodekloud.com>
Date:   Tue Sep 26 09:01:11 2023 +0000

    add data.txt file

commit 8e3fef7312b44bc9b29d9b7ee4f62a3cb4649ff9
Author: Admin <admin@kodekloud.com>
Date:   Tue Sep 26 09:01:11 2023 +0000

```

We are done.
