# Create Folder In Jenkins

## Task

> The DevOps team of xFusionCorp Industries is planning to create a number of Jenkins jobs for different tasks. To easily manage the jobs within Jenkins UI they decided to create different Folders for all Jenkins jobs based on usage/nature of these jobs. Based on the requirements shared below please perform the below mentioned task:
>
> Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.
>
> * There are already two jobs `httpd-php` and `services`.
> * Create a new folder called `Apache` under Jenkins UI.
> * Move the above mentioned two jobs under Apache folder.
>
> **Notes:**
>
> * You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also, in case Jenkins UI gets stuck when Jenkins service restarts in the back end, please make sure to refresh the UI page.


## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Folders
  * https://plugins.jenkins.io/cloudbees-folder/
  * https://docs.cloudbees.com/docs/cloudbees-ci/latest/cloud-secure-guide/folders
* Managing Jenkins Plugins
  * https://www.jenkins.io/doc/book/managing/plugins/


## Steps

* Click the Jenkins button to access to UI through a broswer.
* Login with the `admin` credentials.
* Click Manage Jenkins > Manage Users > Create User and create the user.
* Follow https://www.jenkins.io/doc/book/managing/plugins/#from-the-web-ui and install the Folders plugin.
* After Jenkins restarts, login with the `admin` credentials.
* Click the + button and create the `Apache` folder.
* Click each job and move them into the new folder.

We are done.
