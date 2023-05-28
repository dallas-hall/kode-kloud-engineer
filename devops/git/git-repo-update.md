# Git Repo Update

## Task

> The Nautilus development team started with new project development. They have created different Git repositories to manage respective project's source code. One of the repo `/opt/demo.git` was created recently. The team has given us a sample `index.html` file that is currently present on jump host under `/tmp`. The repository has been cloned at` /usr/src/kodekloudrepos` on storage server in Stratos DC.<br><br>Copy sample `index.html` file from jump host to storage server under cloned repository at `/usr/src/kodekloudrepos`, add/commit the file and push to master branch.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

None.

## Steps

```bash
# Check the cloned repo.
ssh natasha@ststor01 ls -Ahl /usr/src/kodekloudrepos
```

```
natasha@ststor01's password:
total 4.0K
drwxr-xr-x 3 root root 4.0K Mar 10 04:15 demo
```

```bash
# Copy the file over
scp /tmp/index.html natasha@ststor01:/usr/src/kodekloudrepos/demo/
```

```
natasha@ststor01's password:
scp: /usr/src/kodekloudrepos/demo//index.html: Permission denied
```

```bash
# Copy the file over to another place we have permission to
scp /tmp/index.html natasha@ststor01:/tmp
```

```
natasha@ststor01's password:
index.html
```

```bash
# Connect to the storage server
ssh natasha@ststor01

# Switch to root
sudo su

# Copy the file, it will automatically get deleted later
cp /tmp/index.html /usr/src/kodekloudrepos/demo/

# Commit the copied file
cd /usr/src/kodekloudrepos/demo/
git add .
git commit -m 'Adding index.html'
```

```
[master 4bae432] Adding index.html
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
```

```bash
# Push to master
git push origin master
```

```
Counting objects: 4, done.
Delta compression using up to 36 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 335 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To /opt/demo.git
   c90567b..4bae432  master -> master
```

We are done.
