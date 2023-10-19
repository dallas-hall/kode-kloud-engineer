# Install Jenkins

## Task

> The DevOps team of xFusionCorp Industries is planning to setup some CI/CD pipelines. After several meetings they have decided to use Jenkins server. So, we need to setup a Jenkins Server as soon as possible. Please complete the task as per requirements mentioned below:
>
> * Install jenkins on jenkins server using `yum` utility only, and start its service. You might face timeout issue while starting the Jenkins service, please refer this link for help.
> * Jenkin's admin user name should be `theadmin`, password should be `Adm!n321`, full name should be `Anita` and email should be `anita@jenkins.stratos.xfusioncorp.com`.
>
> For this task, ssh into the jenkins server using user `root` and password `S3curePass` from jump host. After installing the Jenkins server, please click on the Jenkins button on the top bar to access Jenkins UI and follow the on-screen instructions to create an admin user.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Installing Jenkins
  * https://www.jenkins.io/doc/book/installing/linux/#red-hat-centos
* Starting Jenkins
  * https://www.jenkins.io/doc/book/system-administration/systemd-services/#starting-services

## Steps

```bash
# Connect to the jenkins server
ssh root@jenkins

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Install wget
dnf install -y wget

# Download the Jenkins yum repo.
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

# Import the Jenkins key
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Update the system
dnf upgrade -y

# Install Jenkins & dependencies
dnf install -y jenkins fontconfig java-17-openjdk

# Reload systemd services and then enable and start Jenkins.
systemctl daemon-reload
systemctl enable --now jenkins

# Log into Jenkins UI in a browser and complete the setup.
cat /var/lib/jenkins/secrets/initialAdminPassword
```

We are done.

