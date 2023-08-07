# Linux Archives With Tar & Gzip

## Task

> On Nautilus storage server in Stratos DC, there is a storage location named `/data`, which is used by different developers to keep their data (non confidential data). One of the developers named `james` has raised a ticket and asked for a copy of their data present in `/data/james` directory on storage server. `/home` is a FTP location on storage server itself where developers can download their data. Below are the instructions shared by the system admin team to accomplish this task.<br>a. Make a `james.tar.gz` compressed archive of `/data/james` directory and move the archive to `/home` directory on Storage Server.



## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None

## Steps

```bash
# Connect to storage server.
ssh natasha@ststor01

# Switch to root
sudo -i

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Change to /data and make the backup
cd /data
tar zcvf james.tar.gz james/
```

```
james/
james/nautilus2.txt
james/nautilus3.txt
james/nautilus1.txt
```

```bash
# Move the backup to /home
mv james.tar.gz /home

# Check it was moved
ls -Ahl --color /home
```

```
...
-rw-r--r-- 1 root    root     186 Aug  1 08:31 james.tar.gz
...
```

We are done.