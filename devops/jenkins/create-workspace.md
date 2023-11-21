# Create Workspace In Jenkins

## Task

> Some developers are working on a common repository where they are testing some features for an application. They are having three branches (excluding the master branch) in this repository where they are adding changes related to these different features. They want to test these changes on Stratos DC app servers so they need a Jenkins job using which they can deploy these different branches as per requirements. Configure a Jenkins job accordingly.
>
> Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.
>
> Similarly, click on Gitea button to access the Gitea page. Login to Gitea server using username `sarah` and password `Sarah_pass123`
>
> * There is a Git repository named `web_app` on Gitea where developers are pushing their changes. It has three branches `version1`, `version2` and `version3` (excluding the `master` branch). You need not to make any changes in the repository.
> * Create a Jenkins job named `app-job`.
> * Configure this job to have a choice parameter named `Branch` with choices as given below:
>   * version1
>   * version2
>   * version3
> * Configure the job to fetch changes from above mentioned Git repository and make sure it should fetches the changes from the respective branch which you are passing as a choice in the choice parameter while building the job. For example if you choose version1 then it must fetch and deploy the changes from branch version1.
> * Configure this job to use custom workspace rather than a default workspace and custom workspace directory should be created under `/var/lib/jenkins` (for example `/var/lib/jenkins/version1`) location rather than under any sub-directory etc. The job should use a workspace as per the value you will pass for Branch parameter while building the job. For example if you choose version1 while building the job then it should create a workspace directory called version1 and should fetch Git repository etc within that directory only.
> * Configure the job to deploy code (fetched from Git repository) on storage server (in Stratos DC) under `/var/www/html` directory. Since its a shared volume. You can access the website by clicking on App button.

>
> **Notes:**
>
> * You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also, in case Jenkins UI gets stuck when Jenkins service restarts in the back end, please make sure to refresh the UI page.


## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create a workspace
  * https://www.jenkins.io/doc/pipeline/steps/workflow-basic-steps/
  * https://community.jenkins.io/t/workspace-creation-in-jenkins-ui/6734
  * https://stackoverflow.com/questions/39397329/jenkins-project-artifacts-and-workspace
* Create a job
  * https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm
* `sshpass`
  * https://stackoverflow.com/a/13955428
  * https://stackoverflow.com/a/16734873
* `scp`
  * https://serverfault.com/a/330504

## Steps

* Click the Gitea button to access to UI through a broswer.
* Login with the `sarah` credentials.
* Navigate to the repository and do a http clone.
* Click the Jenkins button to access to UI through a broswer.
* Login with the `admin` credentials.
* Follow https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm to create the job listed above.
  * Add the following shell script.

```bash
# Variables
export SARAH_PASSWORD='Sarah_pass123'
export NATASHA_PASSWORD='Bl@kW'
export WORKSPACE="/var/lib/jenkins/$Branch"
export APP='web_app'

# Create workspace
rm -rf "$WORKSPACE"
mkdir -p "$WORKSPACE"
cd "$WORKSPACE"

# Clone code
git clone "http://sarah:$SARAH_PASSWORD@git.stratos.xfusioncorp.com/sarah/web_app.git"
cd "$APP"
git checkout "$Branch"

# Deploy code
sshpass -p "$NATASHA_PASSWORD" ssh -o StrictHostKeyChecking=no natasha@ststor01.stratos.xfusioncorp.com "echo $NATASHA_PASSWORD | sudo -S rm -rf /tmp/$APP"
sshpass -p "$NATASHA_PASSWORD" scp -o StrictHostKeyChecking=no -r /var/lib/jenkins/$Branch/web_app natasha@ststor01.stratos.xfusioncorp.com:/tmp/$APP
sshpass -p "$NATASHA_PASSWORD" ssh -o StrictHostKeyChecking=no natasha@ststor01.stratos.xfusioncorp.com "echo $NATASHA_PASSWORD | sudo -S mv /tmp/$APP/* /var/www/html"
```

* Trigger the job.

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/app-job
[app-job] $ /bin/sh -xe /tmp/jenkins394400579724882228.sh
+ export SARAH_PASSWORD='Sarah_pass123'
+ export NATASHA_PASSWORD=Bl@kW
+ export WORKSPACE=/var/lib/jenkins/version1
+ export APP=web_app
+ rm -rf /var/lib/jenkins/version1
+ mkdir -p /var/lib/jenkins/version1
+ cd /var/lib/jenkins/version1
+ git clone http://sarah:Sarah_pass123@git.stratos.xfusioncorp.com/sarah/web_app.git
Cloning into 'web_app'...
+ cd web_app
+ git checkout version1
Switched to a new branch 'version1'
Branch 'version1' set up to track remote branch 'version1' from 'origin'.
+ sshpass -p Bl@kW ssh -o StrictHostKeyChecking=no natasha@ststor01.stratos.xfusioncorp.com echo Bl@kW | sudo -S rm -rf /tmp/web_app
+ sshpass -p Bl@kW scp -o StrictHostKeyChecking=no -r /var/lib/jenkins/version1/web_app natasha@ststor01.stratos.xfusioncorp.com:/tmp/web_app
+ sshpass -p Bl@kW ssh -o StrictHostKeyChecking=no natasha@ststor01.stratos.xfusioncorp.com echo Bl@kW | sudo -S mv /tmp/web_app/* /var/www/html
[sudo] password for natasha: Finished: SUCCESS
```

</details>

Click the App button to view the deployed application.

We are done.
