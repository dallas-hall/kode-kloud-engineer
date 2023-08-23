# Change Maximum Processes For A User

## Task

> On our Storage server in Stratos Datacenter we are having some issues where `nfsuser` user is holding hundred of processes, which is degrading the performance of the server. Therefore, we have a requirement to limit its maximum processes. Please set its maximum process limits as below:<br><br>a. `soft limit` = 1024<br>b. `hard_limit` = 2024

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Process limits:
  * https://serverfault.com/questions/637212/increasing-ulimit-on-centos
  * https://access.redhat.com/solutions/61334

## Steps

```bash
# Connect to the storage server.
ssh natasha@ststor01

# Get root
sudo -i

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Create the new ulimits file
cat > /etc/security/limits.d/90-nproc.conf
```

```
nfsuser soft  nproc 1024
nfsuser hard  nproc 2024
```

Close the file with conttrol + d i.e. `^D`.

```bash
# Reload config file.
sysctl -p
```

We are done.
