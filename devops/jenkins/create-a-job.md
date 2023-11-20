# Create A Jenkins Job

## Task

> Some new requirements have come up to install and configure some packages on the Nautilus infrastructure under Stratos Datacenter. The Nautilus DevOps team installed and configured a new Jenkins server so they wanted to create a Jenkins job to automate this task. Find below more details and complete the task accordingly:
>
> Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username `admin` and `Adm!n321` password.
>
> * Create a Jenkins job named `install-packages` and configure it to accomplish below given tasks.
>   * Add a string parameter named `PACKAGE`.
>   * Configure it to install a package on the storage server in Stratos Datacenter provided to the `$PACKAGE` parameter.
>
> **Note:** Once you restart Jenkins service it will go down for some time so please wait for the Jenkins login page to come back before clicking on the Check button.  For these scenarios requiring changes to be done in a web UI, please take screenshots so you can share them with us for review in case your task is marked incomplete. You may also consider using screen recording software like loom.com to record and share your work

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create a job
  * https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm
* `sshpass`
  * https://stackoverflow.com/a/16734873
* `sudo -S`
  * https://superuser.com/a/67766


## Steps

* Click the Jenkins button to access to UI through a broswer.
* Login with the credentials.
* Follow https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm to create the job listed above.
  * Tick This project is parameterized and add `PACKAGE`
  * Excute a shell script in the build steps with the contents:

```bash
sshpass -p "$NATASHA_PASSWORD" ssh -o StrictHostKeyChecking=no natasha@ststor01.stratos.xfusioncorp.com "echo $NATASHA_PASSWORD | sudo -S yum install -y $PACKAGE"
```

* Save the changes and run the job with Build With Parameters. Provide a package name like `nginx`
* Checking the console output showed:

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/install-packages
[install-packages] $ /bin/sh -xe /tmp/jenkins16823324662058385868.sh
+ sshpass -p Bl@kW ssh -o StrictHostKeyChecking=no natasha@ststor01.stratos.xfusioncorp.com echo Bl@kW | sudo -S yum install -y nginx

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for natasha: Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered with an entitlement server. You can use subscription-manager to register.

CentOS Stream 8 - AppStream                      10 kB/s | 4.4 kB     00:00    
CentOS Stream 8 - AppStream                      30 MB/s |  34 MB     00:01    
CentOS Stream 8 - BaseOS                         13 kB/s | 3.9 kB     00:00    
CentOS Stream 8 - BaseOS                         41 MB/s |  53 MB     00:01    
CentOS Stream 8 - Extras                        2.2 kB/s | 2.9 kB     00:01    
CentOS Stream 8 - Extras common packages        5.1 kB/s | 3.0 kB     00:00    
CentOS Stream 8 - Extras common packages         18 kB/s | 6.9 kB     00:00    
Extra Packages for Enterprise Linux 8 - x86_64   14 MB/s |  16 MB     00:01    
Extra Packages for Enterprise Linux Modular 8 - 1.1 MB/s | 733 kB     00:00    
Red Hat Universal Base Image 8 (RPMs) - BaseOS  4.2 MB/s | 717 kB     00:00    
Red Hat Universal Base Image 8 (RPMs) - AppStre  15 MB/s | 3.0 MB     00:00    
Red Hat Universal Base Image 8 (RPMs) - CodeRea 903 kB/s | 103 kB     00:00    
Dependencies resolved.
==========================================================================================================
 Package                       Arch    Version                                 Repository             Size
==========================================================================================================
Installing:
 nginx                         x86_64  1:1.14.1-9.module+el8.0.0+4108+af250afe ubi-8-appstream-rpms  570 k
Upgrading:
 openssl-libs                  x86_64  1:1.1.1k-9.el8_7                        ubi-8-baseos-rpms     1.5 M
Installing dependencies:
 dejavu-fonts-common           noarch  2.35-7.el8                              baseos                 74 k
 dejavu-sans-fonts             noarch  2.35-7.el8                              baseos                1.6 M
 fontconfig                    x86_64  2.13.1-4.el8                            baseos                274 k
 fontpackages-filesystem       noarch  1.44-22.el8                             baseos                 16 k
 freetype                      x86_64  2.9.1-9.el8                             baseos                394 k
 gd                            x86_64  2.2.5-7.el8                             appstream             144 k
 jbigkit-libs                  x86_64  2.1-14.el8                              appstream              55 k
 libX11                        x86_64  1.6.8-7.el8                             appstream             612 k
 libX11-common                 noarch  1.6.8-7.el8                             appstream             211 k
 libXau                        x86_64  1.0.9-3.el8                             appstream              37 k
 libXpm                        x86_64  3.5.12-11.el8                           appstream              59 k
 libjpeg-turbo                 x86_64  1.5.3-12.el8                            appstream             157 k
 libpng                        x86_64  2:1.6.34-5.el8                          baseos                126 k
 libtiff                       x86_64  4.0.9-29.el8_8                          ubi-8-appstream-rpms  189 k
 libwebp                       x86_64  1.0.0-9.el8_9.1                         ubi-8-appstream-rpms  274 k
 libxcb                        x86_64  1.13.1-1.el8                            appstream             229 k
 libxslt                       x86_64  1.1.32-6.el8                            baseos                250 k
 nginx-all-modules             noarch  1:1.14.1-9.module+el8.0.0+4108+af250afe ubi-8-appstream-rpms   24 k
 nginx-filesystem              noarch  1:1.14.1-9.module+el8.0.0+4108+af250afe ubi-8-appstream-rpms   24 k
 nginx-mod-http-image-filter   x86_64  1:1.14.1-9.module+el8.0.0+4108+af250afe ubi-8-appstream-rpms   35 k
 nginx-mod-http-perl           x86_64  1:1.14.1-9.module+el8.0.0+4108+af250afe ubi-8-appstream-rpms   46 k
 nginx-mod-http-xslt-filter    x86_64  1:1.14.1-9.module+el8.0.0+4108+af250afe ubi-8-appstream-rpms   34 k
 nginx-mod-mail                x86_64  1:1.14.1-9.module+el8.0.0+4108+af250afe ubi-8-appstream-rpms   64 k
 nginx-mod-stream              x86_64  1:1.14.1-9.module+el8.0.0+4108+af250afe ubi-8-appstream-rpms   85 k
 openssl                       x86_64  1:1.1.1k-9.el8_7                        ubi-8-baseos-rpms     710 k
Installing weak dependencies:
 openssl-pkcs11                x86_64  0.4.10-3.el8                            baseos                 66 k
Enabling module streams:
 nginx                                 1.14                                                               

Transaction Summary
==========================================================================================================
Install  27 Packages
Upgrade   1 Package

Total download size: 7.7 M
Downloading Packages:
(1/28): jbigkit-libs-2.1-14.el8.x86_64.rpm      348 kB/s |  55 kB     00:00    
(2/28): gd-2.2.5-7.el8.x86_64.rpm               740 kB/s | 144 kB     00:00    
(3/28): libXau-1.0.9-3.el8.x86_64.rpm           926 kB/s |  37 kB     00:00    
(4/28): libX11-1.6.8-7.el8.x86_64.rpm           2.2 MB/s | 612 kB     00:00    
(5/28): libX11-common-1.6.8-7.el8.noarch.rpm    1.7 MB/s | 211 kB     00:00    
(6/28): libXpm-3.5.12-11.el8.x86_64.rpm         1.2 MB/s |  59 kB     00:00    
(7/28): libjpeg-turbo-1.5.3-12.el8.x86_64.rpm   3.6 MB/s | 157 kB     00:00    
(8/28): libxcb-1.13.1-1.el8.x86_64.rpm          5.2 MB/s | 229 kB     00:00    
(9/28): dejavu-fonts-common-2.35-7.el8.noarch.r 330 kB/s |  74 kB     00:00    
(10/28): fontpackages-filesystem-1.44-22.el8.no 327 kB/s |  16 kB     00:00    
(11/28): fontconfig-2.13.1-4.el8.x86_64.rpm     991 kB/s | 274 kB     00:00    
(12/28): libpng-1.6.34-5.el8.x86_64.rpm         2.1 MB/s | 126 kB     00:00    
(13/28): dejavu-sans-fonts-2.35-7.el8.noarch.rp 4.0 MB/s | 1.6 MB     00:00    
(14/28): freetype-2.9.1-9.el8.x86_64.rpm        2.4 MB/s | 394 kB     00:00    
(15/28): libxslt-1.1.32-6.el8.x86_64.rpm        3.4 MB/s | 250 kB     00:00    
(16/28): openssl-pkcs11-0.4.10-3.el8.x86_64.rpm 1.4 MB/s |  66 kB     00:00    
(17/28): nginx-all-modules-1.14.1-9.module+el8. 445 kB/s |  24 kB     00:00    
(18/28): nginx-filesystem-1.14.1-9.module+el8.0 1.4 MB/s |  24 kB     00:00    
(19/28): nginx-1.14.1-9.module+el8.0.0+4108+af2 5.6 MB/s | 570 kB     00:00    
(20/28): openssl-1.1.1k-9.el8_7.x86_64.rpm      5.5 MB/s | 710 kB     00:00    
(21/28): nginx-mod-http-image-filter-1.14.1-9.m 1.8 MB/s |  35 kB     00:00    
(22/28): nginx-mod-http-perl-1.14.1-9.module+el 2.5 MB/s |  46 kB     00:00    
(23/28): nginx-mod-mail-1.14.1-9.module+el8.0.0 2.6 MB/s |  64 kB     00:00    
(24/28): nginx-mod-stream-1.14.1-9.module+el8.0 4.7 MB/s |  85 kB     00:00    
(25/28): libwebp-1.0.0-9.el8_9.1.x86_64.rpm      14 MB/s | 274 kB     00:00    
(26/28): nginx-mod-http-xslt-filter-1.14.1-9.mo 629 kB/s |  34 kB     00:00    
(27/28): libtiff-4.0.9-29.el8_8.x86_64.rpm      5.3 MB/s | 189 kB     00:00    
(28/28): openssl-libs-1.1.1k-9.el8_7.x86_64.rpm  35 MB/s | 1.5 MB     00:00    
--------------------------------------------------------------------------------
Total                                           6.6 MB/s | 7.7 MB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : libpng-2:1.6.34-5.el8.x86_64                          1/29 
  Installing       : freetype-2.9.1-9.el8.x86_64                           2/29 
  Installing       : fontpackages-filesystem-1.44-22.el8.noarch            3/29 
  Installing       : libjpeg-turbo-1.5.3-12.el8.x86_64                     4/29 
  Installing       : dejavu-fonts-common-2.35-7.el8.noarch                 5/29 
  Installing       : dejavu-sans-fonts-2.35-7.el8.noarch                   6/29 
  Installing       : fontconfig-2.13.1-4.el8.x86_64                        7/29 
  Running scriptlet: fontconfig-2.13.1-4.el8.x86_64                        7/29 
  Installing       : openssl-1:1.1.1k-9.el8_7.x86_64                       8/29 
  Upgrading        : openssl-libs-1:1.1.1k-9.el8_7.x86_64                  9/29 
  Running scriptlet: openssl-libs-1:1.1.1k-9.el8_7.x86_64                  9/29 
  Installing       : openssl-pkcs11-0.4.10-3.el8.x86_64                   10/29 
  Installing       : libwebp-1.0.0-9.el8_9.1.x86_64                       11/29 
  Running scriptlet: nginx-filesystem-1:1.14.1-9.module+el8.0.0+4108+af   12/29 
  Installing       : nginx-filesystem-1:1.14.1-9.module+el8.0.0+4108+af   12/29 
  Installing       : libxslt-1.1.32-6.el8.x86_64                          13/29 
  Installing       : libXau-1.0.9-3.el8.x86_64                            14/29 
  Installing       : libxcb-1.13.1-1.el8.x86_64                           15/29 
  Installing       : libX11-common-1.6.8-7.el8.noarch                     16/29 
  Installing       : libX11-1.6.8-7.el8.x86_64                            17/29 
  Installing       : libXpm-3.5.12-11.el8.x86_64                          18/29 
  Installing       : jbigkit-libs-2.1-14.el8.x86_64                       19/29 
  Running scriptlet: jbigkit-libs-2.1-14.el8.x86_64                       19/29 
  Installing       : libtiff-4.0.9-29.el8_8.x86_64                        20/29 
  Installing       : gd-2.2.5-7.el8.x86_64                                21/29 
  Running scriptlet: gd-2.2.5-7.el8.x86_64                                21/29 
  Installing       : nginx-mod-http-perl-1:1.14.1-9.module+el8.0.0+4108   22/29 
  Running scriptlet: nginx-mod-http-perl-1:1.14.1-9.module+el8.0.0+4108   22/29 
  Installing       : nginx-mod-http-xslt-filter-1:1.14.1-9.module+el8.0   23/29 
  Running scriptlet: nginx-mod-http-xslt-filter-1:1.14.1-9.module+el8.0   23/29 
  Installing       : nginx-mod-mail-1:1.14.1-9.module+el8.0.0+4108+af25   24/29 
  Running scriptlet: nginx-mod-mail-1:1.14.1-9.module+el8.0.0+4108+af25   24/29 
  Installing       : nginx-mod-stream-1:1.14.1-9.module+el8.0.0+4108+af   25/29 
  Running scriptlet: nginx-mod-stream-1:1.14.1-9.module+el8.0.0+4108+af   25/29 
  Installing       : nginx-1:1.14.1-9.module+el8.0.0+4108+af250afe.x86_   26/29 
  Running scriptlet: nginx-1:1.14.1-9.module+el8.0.0+4108+af250afe.x86_   26/29 
  Installing       : nginx-mod-http-image-filter-1:1.14.1-9.module+el8.   27/29 
  Running scriptlet: nginx-mod-http-image-filter-1:1.14.1-9.module+el8.   27/29 
  Installing       : nginx-all-modules-1:1.14.1-9.module+el8.0.0+4108+a   28/29 
  Cleanup          : openssl-libs-1:1.1.1k-7.el8_6.x86_64                 29/29 
  Running scriptlet: openssl-libs-1:1.1.1k-7.el8_6.x86_64                 29/29 
  Running scriptlet: fontconfig-2.13.1-4.el8.x86_64                       29/29 
  Verifying        : gd-2.2.5-7.el8.x86_64                                 1/29 
  Verifying        : jbigkit-libs-2.1-14.el8.x86_64                        2/29 
  Verifying        : libX11-1.6.8-7.el8.x86_64                             3/29 
  Verifying        : libX11-common-1.6.8-7.el8.noarch                      4/29 
  Verifying        : libXau-1.0.9-3.el8.x86_64                             5/29 
  Verifying        : libXpm-3.5.12-11.el8.x86_64                           6/29 
  Verifying        : libjpeg-turbo-1.5.3-12.el8.x86_64                     7/29 
  Verifying        : libxcb-1.13.1-1.el8.x86_64                            8/29 
  Verifying        : dejavu-fonts-common-2.35-7.el8.noarch                 9/29 
  Verifying        : dejavu-sans-fonts-2.35-7.el8.noarch                  10/29 
  Verifying        : fontconfig-2.13.1-4.el8.x86_64                       11/29 
  Verifying        : fontpackages-filesystem-1.44-22.el8.noarch           12/29 
  Verifying        : freetype-2.9.1-9.el8.x86_64                          13/29 
  Verifying        : libpng-2:1.6.34-5.el8.x86_64                         14/29 
  Verifying        : libxslt-1.1.32-6.el8.x86_64                          15/29 
  Verifying        : openssl-pkcs11-0.4.10-3.el8.x86_64                   16/29 
  Verifying        : openssl-1:1.1.1k-9.el8_7.x86_64                      17/29 
  Verifying        : nginx-1:1.14.1-9.module+el8.0.0+4108+af250afe.x86_   18/29 
  Verifying        : nginx-all-modules-1:1.14.1-9.module+el8.0.0+4108+a   19/29 
  Verifying        : nginx-filesystem-1:1.14.1-9.module+el8.0.0+4108+af   20/29 
  Verifying        : nginx-mod-http-image-filter-1:1.14.1-9.module+el8.   21/29 
  Verifying        : nginx-mod-http-perl-1:1.14.1-9.module+el8.0.0+4108   22/29 
  Verifying        : nginx-mod-http-xslt-filter-1:1.14.1-9.module+el8.0   23/29 
  Verifying        : nginx-mod-mail-1:1.14.1-9.module+el8.0.0+4108+af25   24/29 
  Verifying        : nginx-mod-stream-1:1.14.1-9.module+el8.0.0+4108+af   25/29 
  Verifying        : libtiff-4.0.9-29.el8_8.x86_64                        26/29 
  Verifying        : libwebp-1.0.0-9.el8_9.1.x86_64                       27/29 
  Verifying        : openssl-libs-1:1.1.1k-9.el8_7.x86_64                 28/29 
  Verifying        : openssl-libs-1:1.1.1k-7.el8_6.x86_64                 29/29 
Installed products updated.

Upgraded:
  openssl-libs-1:1.1.1k-9.el8_7.x86_64                                          
Installed:
  dejavu-fonts-common-2.35-7.el8.noarch                                         
  dejavu-sans-fonts-2.35-7.el8.noarch                                           
  fontconfig-2.13.1-4.el8.x86_64                                                
  fontpackages-filesystem-1.44-22.el8.noarch                                    
  freetype-2.9.1-9.el8.x86_64                                                   
  gd-2.2.5-7.el8.x86_64                                                         
  jbigkit-libs-2.1-14.el8.x86_64                                                
  libX11-1.6.8-7.el8.x86_64                                                     
  libX11-common-1.6.8-7.el8.noarch                                              
  libXau-1.0.9-3.el8.x86_64                                                     
  libXpm-3.5.12-11.el8.x86_64                                                   
  libjpeg-turbo-1.5.3-12.el8.x86_64                                             
  libpng-2:1.6.34-5.el8.x86_64                                                  
  libtiff-4.0.9-29.el8_8.x86_64                                                 
  libwebp-1.0.0-9.el8_9.1.x86_64                                                
  libxcb-1.13.1-1.el8.x86_64                                                    
  libxslt-1.1.32-6.el8.x86_64                                                   
  nginx-1:1.14.1-9.module+el8.0.0+4108+af250afe.x86_64                          
  nginx-all-modules-1:1.14.1-9.module+el8.0.0+4108+af250afe.noarch              
  nginx-filesystem-1:1.14.1-9.module+el8.0.0+4108+af250afe.noarch               
  nginx-mod-http-image-filter-1:1.14.1-9.module+el8.0.0+4108+af250afe.x86_64    
  nginx-mod-http-perl-1:1.14.1-9.module+el8.0.0+4108+af250afe.x86_64            
  nginx-mod-http-xslt-filter-1:1.14.1-9.module+el8.0.0+4108+af250afe.x86_64     
  nginx-mod-mail-1:1.14.1-9.module+el8.0.0+4108+af250afe.x86_64                 
  nginx-mod-stream-1:1.14.1-9.module+el8.0.0+4108+af250afe.x86_64               
  openssl-1:1.1.1k-9.el8_7.x86_64                                               
  openssl-pkcs11-0.4.10-3.el8.x86_64                                            

Complete!
Finished: SUCCESS
```

</details>

We are done.
