# Install Tomcat

## Task

> The Nautilus application development team recently finished the beta version of one of their Java-based applications, which they are planning to deploy on one of the app servers in Stratos DC. After an internal team meeting, they have decided to use the tomcat application server. Based on the requirements mentioned below complete the task:
> 
> * Install tomcat server on App Server 1 using yum.
> * Configure it to run on port `6300`.
> * There is a `ROOT.war` file on Jump host at location `/tmp`.
> 
> Deploy it on this tomcat server and make sure the webpage works directly on base URL i.e without specifying any sub-directory anything like this `curl http://stapp01:6300`

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Install Tomcat
  * https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-7-on-centos-7-via-yum
* Change Tomcat port
  * https://stackoverflow.com/questions/18415578/how-to-change-tomcat-port-number
* Copy WAR file to the correct place
  * https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-7-on-centos-7-via-yum

## Steps

```bash
# Copy over our WAR file from the thor jumpbox
scp /tmp/ROOT.war tony@stapp01:/home/tony
```

```
tony@stapp01's password: 
ROOT.war                                        100% 4529    11.9MB/s   00:00
```

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

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
dnf install -y tomcat
```

```
...
Installed:
...
  java-1.8.0-openjdk-headless-1:1.8.0.392.b08-4.el8.x86_64                 
...                                         
  tomcat-1:9.0.62-27.el8.noarch                                            
...
```

```bash
# Update port to 6300 after taking a backup
cp -a /etc/tomcat/server.xml /etc/tomcat/server.xml.old
sed -i -r 's/<Connector port="8080"/<Connector port="6300"/g' /etc/tomcat/server.xml

# Start and enable the Tomcat service
systemctl enable --now tomcat

# Copy the WAR file to the correct place
mv ~tony/ROOT.war /usr/share/tomcat/webapps

# Update WAR ownership
chown tomcat: /usr/share/tomcat/webapps/ROOT.war

# Test webapp
curl -Lv localhost:6300
```

```
 Rebuilt URL to: localhost:6300/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 6300 (#0)
> GET / HTTP/1.1
> Host: localhost:6300
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 200 
< Accept-Ranges: bytes
< ETag: W/"471-1580289830000"
< Last-Modified: Wed, 29 Jan 2020 09:23:50 GMT
< Content-Type: text/html
< Content-Length: 471
< Date: Wed, 06 Dec 2023 04:03:18 GMT
< 
<!DOCTYPE html>
<!--
To change this license header, choose License Headers in Project Properties.
To change this template file, choose Tools | Templates
and open the template in the editor.
-->
<html>
    <head>
        <title>SampleWebApp</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </head>
    <body>
        <h2>Welcome to xFusionCorp Industries!</h2>
        <br>
    
    </body>
</html>
* Connection #0 to host localhost left intact
```

We are done.
