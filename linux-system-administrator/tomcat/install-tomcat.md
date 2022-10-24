# Install Tomcat

## Task

> The Nautilus application development team recently finished the beta version of one of their Java-based applications, which they are planning to deploy on one of the app servers in Stratos DC. After an internal team meeting, they have decided to use the tomcat application server. Based on the requirements mentioned below complete the task:<br><br>a. Install tomcat server on App Server 3 using yum.<br>b. Configure it to run on port `3000`.<br>c. There is a `ROOT.war` file on Jump host at location `/tmp`. Deploy it on this tomcat server and make sure the webpage works directly on base URL i.e without specifying any sub-directory anything like this `http://URL/ROOT`

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Install Tomcat - https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-7-on-centos-7-via-yum
* Change Tomcat port - https://stackoverflow.com/questions/18415578/how-to-change-tomcat-port-number
* Copy WAR file to the correct place - https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-7-on-centos-7-via-yum

## Steps

```bash
# Copy over our WAR file from the thor jumpbox
scp /tmp/ROOT.war banner@stapp03:/home/banner
```

```
banner@stapp03's password: 
ROOT.war                                        100% 4529    11.9MB/s   00:00
```

```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Check the current Java verion
java -version
```

```
-bash: java: command not found
```


```bash
# Install Tomcat which installs Java 1.8
yum install -y tomcat
```

```
...
Installed:
  tomcat.noarch 0:7.0.76-16.el7_9
...
Dependency Installed:
...
  java-1.8.0-openjdk.x86_64 1:1.8.0.345.b01-1.el7_9
...
```

```bash
# Update port to 3000 after taking a backup
cp -a /etc/tomcat/server.xml /etc/tomcat/server.xml.old
sed -i -r 's/<Connector port="8080"/<Connector port="3000"/g' /etc/tomcat/server.xml

# Start and enable the Tomcat service
systemctl enable --now tomcat
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/tomcat.service to /usr/lib/systemd/system/tomcat.service.
```

```bash
# Copy the WAR file to the correct place
mv ~banner/ROOT.war /usr/share/tomcat/webapps

# Update WAR ownership
chown tomcat: /usr/share/tomcat/webapps/ROOT.war

# Test webapp
curl -Lv localhost:3000/ROOT
```

```
* About to connect() to localhost port 3000 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 3000 (#0)
> GET /ROOT HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:3000
> Accept: */*
> 
< HTTP/1.1 404 Not Found
< Server: Apache-Coyote/1.1
< Content-Type: text/html;charset=utf-8
< Content-Language: en
< Content-Length: 959
< Date: Mon, 24 Oct 2022 01:30:59 GMT
< 
* Connection #0 to host localhost left intact
<html><head><title>Apache Tomcat/7.0.76 - Error report</title><style><!--H1 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:22px;} H2 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:16px;} H3 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:14px;} BODY {font-family:Tahoma,Arial,sans-serif;color:black;background-color:white;} B {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;} P {font-family:Tahoma,Arial,sans-serif;background:white;color:black;font-size:12px;}A {color : black;}A.name {color : black;}HR {color : #525D76;}--></style> </head><body><h1>HTTP Status 404 - /ROOT</h1><HR size="1" noshade="noshade"><p><b>type</b> Status report</p><p><b>message</b> <u>/ROOT</u></p><p><b>description</b> <u>The requested resource is not available.</u></p><HR size="1" noshade="noshade"><h3>Apache Tomcat/7.0.76</h3></body></html>
```
