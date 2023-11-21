# Create A Parametised Jenkins Job

## Task

> A new DevOps Engineer has joined the team and he will be assigned some Jenkins related tasks. Before that, the team wanted to test a simple parameterized job to understand basic functionality of parameterized builds. He is given a simple parameterized job to build in Jenkins. Please find more details below:
>
> Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username `admin` and `Adm!n321` password.
>
> * Create a parameterized job which should be named as `parameterized-job`
> * Add a string parameter named `Stage`; its default value should be `Build`.
> * Add a choice parameter named `env`; its choices should be `Development`, `Staging` and `Production`.
> * Configure job to execute a shell command, which should echo both parameter values (you are passing in the job).
> * Build the Jenkins job at least once with choice parameter value `Staging` to make sure it passes.
>
> **Note:** Once you restart Jenkins service it will go down for some time so please wait for the Jenkins login page to come back before clicking on the Check button.  For these scenarios requiring changes to be done in a web UI, please take screenshots so you can share them with us for review in case your task is marked incomplete. You may also consider using screen recording software like loom.com to record and share your work

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Create a job
  * https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm

## Steps

* Click the Jenkins button to access to UI through a broswer.
* Login with the credentials.
* Follow https://www.tutorialspoint.com/jenkins/jenkins_setup_build_jobs.htm to create the job listed above.
* Run the job and inspect the console output.

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/parameterized-job
[parameterized-job] $ /bin/sh -xe /tmp/jenkins3484539419393012614.sh
+ echo The stage is Build
The stage is Build
+ echo The environment is Staging
The environment is Staging
Finished: SUCCESS
```

</details>

We are done.
