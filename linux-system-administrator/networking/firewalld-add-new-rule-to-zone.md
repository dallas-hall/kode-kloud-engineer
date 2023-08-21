# Add New FirewallD Rule

## Task

> The Nautilus system admins team recently deployed a web UI application for their backup utility running on the *Nautilus backup server* in Stratos Datacenter. The application is running on port 6000. They have `firewalld` installed on that server. The requirements that have come up include the following:<br><br>Open all incoming connection on `6000/tcp` port. Zone should be `public`.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* firewalld
  * https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7
* firewall-cmd
  * https://fedoramagazine.org/control-the-firewall-at-the-command-line/


## Steps

```bash
# Connect to the first app server
ssh clint@stbkp01

# Get root
sudo -i

# Check current Linux version, it was CentOS 7.6
cat /etc/*rel*

# Check the app is running.
ss -lntp
```

```
State      Recv-Q Send-Q                           Local Address:Port                                          Peer Address:Port
...                                         *:*
LISTEN     0      511                                          *:6000                                                     *:*                   users:(("httpd",pid=706,fd=3),("httpd",pid=705,fd=3),("httpd",pid=704,fd=3),("httpd",pid=703,fd=3),("httpd",pid=702,fd=3),("httpd",pid=701,fd=3))
...
```

```bash
# Check firewalld status
systemctl status firewalld
```

```
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2023-08-21 08:45:12 UTC; 5min ago
...
```

```bash
# Check firewall status
firewall-cmd --state
```

```
running
```

```bash
# View firewall zones.
firewall-cmd --get-zones
```

```
block dmz drop external home internal public trusted work
```

```bash
# View default firewall zone.
firewall-cmd --get-default-zone
```

```
public
```

```bash
# View active firewall zones.
firewall-cmd --get-active-zones

# There are none so there was no output

# Add a new rule.
firewall-cmd --add-port=6000/tcp --permanent
```

```
success
```

```bash
# View network interfaces.
ip -c -h a
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
14: eth0@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:10:ee:10 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.16/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
20: eth1@if21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:05 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.5/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```

Use `eth0` because the IP address matches https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

```bash
# Set default zone to public for the network interface.
firewall-cmd --change-interface=eth0 --zone=public
```

```
success
```

```bash
# View active firewall zones.
firewall-cmd --get-active-zones
```

```
public
  interfaces: eth0
```

```bash
# Reload firewalld.
systemctl reload firewalld

# View the open firewall ports.
firewall-cmd --list-ports
```

```
6000/tcp
```

```bash
# Test local connectivity
curl -v localhost:6000
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd"><html><head>
<meta http-equiv="content-type" content="text/html; charset=UTF-8">
                <title>Apache HTTP Server Test Page powered by CentOS</title>
                <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">

    <!-- Bootstrap -->
    <link href="/noindex/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="noindex/css/open-sans.css" type="text/css" />

<style type="text/css"><!--

body {
  font-family: "Open Sans", Helvetica, sans-serif;
  font-weight: 100;
  color: #ccc;
  background: rgba(10, 24, 55, 1);
  font-size: 16px;
}

h2, h3, h4 {
  font-weight: 200;
}

h2 {
  font-size: 28px;
}

.jumbotron {
  margin-bottom: 0;
  color: #333;
  background: rgb(212,212,221); /* Old browsers */
  background: radial-gradient(ellipse at center top, rgba(255,255,255,1) 0%,rgba(174,174,183,1) 100%); /* W3C */
}

.jumbotron h1 {
  font-size: 128px;
  font-weight: 700;
  color: white;
  text-shadow: 0px 2px 0px #abc,
               0px 4px 10px rgba(0,0,0,0.15),
               0px 5px 2px rgba(0,0,0,0.1),
               0px 6px 30px rgba(0,0,0,0.1);
}

.jumbotron p {
  font-size: 28px;
  font-weight: 100;
}

.main {
   background: white;
   color: #234;
   border-top: 1px solid rgba(0,0,0,0.12);
   padding-top: 30px;
   padding-bottom: 40px;
}

.footer {
   border-top: 1px solid rgba(255,255,255,0.2);
   padding-top: 30px;
}

    --></style>
</head>
<body>
  <div class="jumbotron text-center">
    <div class="container">
          <h1>Testing 123..</h1>
                <p class="lead">This page is used to test the proper operation of the <a href="http://apache.org">Apache HTTP server</a> after it has been installed. If you can read this page it means that this site is working properly. This server is powered by <a href="http://centos.org">CentOS</a>.</p>
                </div>
  </div>
  <div class="main">
    <div class="container">
       <div class="row">
                        <div class="col-sm-6">
                        <h2>Just visiting?</h2>
                                        <p class="lead">The website you just visited is either experiencing problems or is undergoing routine maintenance.</p>
                                        <p>If you would like to let the administrators of this website know that you've seen this page instead of the page you expected, you should send them e-mail. In general, mail sent to the name "webmaster" and directed to the website's domain should reach the appropriate person.</p>
                                        <p>For example, if you experienced problems while visiting www.example.com, you should send e-mail to "webmaster@example.com".</p>
                                </div>
                                <div class="col-sm-6">
                                        <h2>Are you the Administrator?</h2>
                                        <p>You should add your website content to the directory <tt>/var/www/html/</tt>.</p>
                                        <p>To prevent this page from ever being used, follow the instructions in the file <tt>/etc/httpd/conf.d/welcome.conf</tt>.</p>

                                        <h2>Promoting Apache and CentOS</h2>
                                        <p>You are free to use the images below on Apache and CentOS Linux powered HTTP servers.  Thanks for using Apache and CentOS!</p>
                                        <p><a href="http://httpd.apache.org/"><img src="images/apache_pb.gif" alt="[ Powered by Apache ]"></a> <a href="http://www.centos.org/"><img src="images/poweredby.png" alt="[ Powered by CentOS Linux ]" height="31" width="88"></a></p>
                                </div>
                        </div>
            </div>
                </div>
        </div>
          <div class="footer">
      <div class="container">
        <div class="row">
          <div class="col-sm-6">
            <h2>Important note:</h2>
            <p class="lead">The CentOS Project has nothing to do with this website or its content,
            it just provides the software that makes the website run.</p>

            <p>If you have issues with the content of this site, contact the owner of the domain, not the CentOS project.
            Unless you intended to visit CentOS.org, the CentOS Project does not have anything to do with this website,
            the content or the lack of it.</p>
            <p>For example, if this website is www.example.com, you would find the owner of the example.com domain at the following WHOIS server:</p>
            <p><a href="http://www.internic.net/whois.html">http://www.internic.net/whois.html</a></p>
          </div>
          <div class="col-sm-6">
            <h2>The CentOS Project</h2>
            <p>The CentOS Linux distribution is a stable, predictable, manageable and reproduceable platform derived from
               the sources of Red Hat Enterprise Linux (RHEL).<p>

            <p>Additionally to being a popular choice for web hosting, CentOS also provides a rich platform for open source communities to build upon. For more information
               please visit the <a href="http://www.centos.org/">CentOS website</a>.</p>
          </div>
        </div>
                  </div>
    </div>
  </div>
</body></html>
```

</details>

```bash
# Log out of root
exit

# Log off from the backup server.
exit

# Check connectivity from the jumphost
curl stbkp01:6000
```

The HTML output is the same as above, we are done.
