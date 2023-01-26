# Git Clone

## Task

> DevOps team created a new Git repository last week; however, as of now no team is using it. The Nautilus application development team recently asked for a copy of that repo on Storage server in Stratos DC. Please clone the repo as per details shared below:<br><br>The repo that needs to be cloned is `/opt/ecommerce.git`<br>Clone this git repository under `/usr/src/kodekloudrepos` directory. Please do not try to make any changes in repo.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

None

## Steps

```bash
# Connect to the storage server
ssh natasha@ststor01

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Go to the correct path
cd /usr/src/kodekloudrepos

# Clone the repo
git clone /opt/ecommerce.git
```

```
Cloning into 'ecommerce'...
warning: You appear to have cloned an empty repository.
done.
```

```bash
# Check the clone
cd ecommerce
git status
```

```
# On branch master
#
# Initial commit
#
nothing to commit (create/copy files and use "git add" to track)
```

We are done.