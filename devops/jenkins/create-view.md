# Create Views In Jenkins

## Task

> The DevOps team of xFusionCorp Industries is planning to create a number of Jenkins jobs for different tasks. So to easily manage the jobs within Jenkins UI they decided to create different views for all Jenkins jobs based on usage/nature of these jobs, - for example `nautilus-crons` view for all cron jobs. Based on the requirements shared below please perform the below mentioned task:
>
> Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.
>
> * Create a Jenkins job named `nautilus-test-job`.
> * Configure this job to run a simple bash command i.e `echo "hello world!!"`.
> * Create a view named `nautilus-crons` (it must be a global view of type List View) and make sure `nautilus-test-job` and `nautilus-cron-job` (which is already present on Jenkins) jobs are listed under this new view.
> * Schedule this newly created job to build periodically at every minute i.e `* * * * *` (please make sure to use the cron expression exactly same how it is mentioned here)
> * Make sure the job builds successfully.
>
> **Notes:**
>
> * You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also, in case Jenkins UI gets stuck when Jenkins service restarts in the back end, please make sure to refresh the UI page.


## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create a view
  * https://stackoverflow.com/questions/32451528/create-a-view-under-jenkins
* Create a job
  * https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm


## Steps

* Click the Jenkins button to access to UI through a broswer.
* Login with the `admin` credentials.
* Follow https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm to create the job listed above.
* Follow https://stackoverflow.com/questions/32451528/create-a-view-under-jenkins to create the view and add the existing jobs into it.
* Trigger the jobs.

We are done.
