# Install httpd & Deploy Basic Applications

## Task

> xFusionCorp Industries is planning to host two static websites on their infra in Stratos Datacenter. The development of these websites is still in-progress, but we want to get the servers ready. Please perform the following steps to accomplish the task:
>
> * Install `httpd` package and dependencies on app server 2.
> * Apache should serve on port `6400`.
> * There are two website's backups `/home/thor/news` and `/home/thor/cluster` on jump_host. Set them up on Apache in a way that news should work on the link http://localhost:6400/news/ and cluster should work on link http://localhost:6400/cluster/ on the mentioned app server.
> * Once configured you should be able to access the website using `curl` command on the respective app server, i.e `curl http://localhost:6400/news/` and `curl http://localhost:6400/cluster/`

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Change listening port
  * https://httpd.apache.org/docs/2.4/en/bind.html

## Steps

```bash
# Copy files from jump host.
tar zcvf /tmp/news.tgz news
tar zcvf /tmp/cluster.tgz cluster
scp /tmp/news.tgz /tmp/cluster.tgz steve@stapp02:/tmp

# Connect to database server
ssh steve@stapp02

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Change to root
sudo -i

# Install useful tools, e.g. ss, nslookup, clear, scp, and required packages
dnf install -y iproute bind-utils ncurses bash-completion openssh-clients  httpd

# Take a backup
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.old

# Change the port httpd is listening on
sed -i -r 's/Listen 80/Listen 6400/g' /etc/httpd/conf/httpd.conf

# Extract the websites.
cd /var/www/html
tar zxvf /tmp/news.tgz
tar zxvf /tmp/cluster.tgz

# Enable and start mariadb
systemctl enable --now httpd

# Test the news website locally.
curl localhost:6400/news/

# Test the news website remotely.
curl stapp02:6400/news/
```

```html
<!DOCTYPE html>
<html>
<body>

<h1>KodeKloud</h1>

<p>This is a sample page for our news website</p>

</body>
</html>
```

```bash
# Test the cluster website locally.
curl localhost:6400/cluster/

# Test the news website remotely.
curl stapp02:6400/cluster/
```

```html
<!DOCTYPE html>
<html>
<body>

<h1>KodeKloud</h1>

<p>This is a sample page for our cluster website</p>

</body>
</html>
```

Both sites are working locally and remotely. We are done.
