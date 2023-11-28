# Git Manage Pull Requests

## Task

> Max want to push some new changes to one of the repositories but we don't want people to push directly to `master` branch, since that would be the final version of the code. It should always only have content that has been reviewed and approved. We cannot just allow everyone to directly push to the `master` branch. So, let's do it the right way as discussed below:
>
> * SSH into storage server using user `max`, password `Max_pass123`. There you can find an already cloned repo under Max user's home.
> * Max has written his story about The 🦊 Fox and Grapes 🍇
> * Max has already pushed his story to remote git repository hosted on Gitea branch `story/fox-and-grapes`
> * Check the contents of the cloned repository. Confirm that you can see Sarah's story and history of commits by running `git log` and validate author info, commit message etc.
> * Max has pushed his story, but his story is still not in the `master` branch. Let's create a Pull Request(PR) to merge Max's `story/fox-and-grapes` branch into the master branch
> * Click on the Gitea UI button on the top bar. You should be able to access the Gitea page.
>   * Username: `max`
>   * Password: `Max_pass123`
>   * PR title : `Added fox-and-grapes story`
>   * PR pull from branch: `story/fox-and-grapes` (source)
>   * PR merge into branch: `master` (destination)
> * Before we can add our story to the `master` branch, it has to be reviewed. So, let's ask `tom` to review our PR by assigning him as a reviewer. Add tom as reviewer through the Git Portal UI
>   * Go to the newly created PR
>   * Click on Reviewers on the right
>   * Add `tom` as a reviewer to the PR
> * Now let's review and approve the PR as user Tom. Login to the portal with the user tom
>   * Username: `tom`
>   * Password: `Tom_pass123`
>   * PR title : `Added fox-and-grapes story`
>   * Review and merge it.
>
> Great stuff!! The story has been merged! 👏

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `git cherry-pick`
  * https://stackoverflow.com/questions/5304391/how-to-cherry-pick-from-1-branch-to-another

## Steps

```bash
# Connect to the storage server
ssh max@ststor01

# View the changes
cd story-blog
git log
```

```
commit ed1404d31e34b31b44f96545d96952eb476701c9
Author: max <max@stratos.xfusioncorp.com>
Date:   Tue Nov 28 00:10:57 2023 +0000

    Added fox-and-grapes story

commit 1c07110b666d339162fdaf5bf24f4113b818cdbe
Merge: a247547 36b103b
Author: sarah <sarah@stratos.xfusioncorp.com>
Date:   Tue Nov 28 00:10:56 2023 +0000

    Merge branch 'story/frogs-and-ox'

commit a247547d2fa065ba5f96f1d3825d1751057fa3ba
Author: sarah <sarah@stratos.xfusioncorp.com>
Date:   Tue Nov 28 00:10:56 2023 +0000

    Fix typo in story title

commit 36b103b3a41d197267013e2510b2a598af8b25fa
Author: sarah <sarah@stratos.xfusioncorp.com>
Date:   Tue Nov 28 00:10:56 2023 +0000

    Completed frogs-and-ox story

commit 2978f59655a278738d5c28593849f2bc9a070145
Author: sarah <sarah@stratos.xfusioncorp.com>
Date:   Tue Nov 28 00:10:56 2023 +0000

    Added the lion and mouse story

commit 6ab7af72d745327810f91b79f34eaa64a53e1354
Author: sarah <sarah@stratos.xfusioncorp.com>
Date:   Tue Nov 28 00:10:56 2023 +0000

    Add incomplete frogs-and-ox story
```

Log into the Gitea UI as Max and create the PR as above.
Log into the Gitea UI as Tom and review and merge the PR as above.

We are done.
