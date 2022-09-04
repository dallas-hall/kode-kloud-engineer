# Copy File With scp

## Task

> One of the Nautilus developers has copied confidential data on the jump host in Stratos DC. That data must be copied to one of the app servers. Because developers do not have access to app servers, they asked the system admins team to accomplish the task for them.<br><br>Copy `/tmp/nautilus.txt.gpg` file from jump server to App Server 3 at location `/home/opt`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* `man scp`

## Steps

```bash
# Copy the files over to app3 server
scp /tmp/nautilus.txt.gpg banner@stapp03:/home/opt
```

```
...
nautilus.txt.gpg                                         100%   74   268.0KB/s   00:00
```

```bash
# Check the file copy
ssh banner@stapp03 ls -Ahl /home/opt
```

```
...
-rw-r--r-- 1 banner banner 74 Aug 28 09:32 nautilus.txt.gpg
```