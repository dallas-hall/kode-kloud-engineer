# Add Slave Nodes

## Task

> The Nautilus DevOps team has installed and configured new Jenkins server in Stratos DC which they will use for CI/CD and for some automation tasks. There is a requirement to add all app servers as slave nodes in Jenkins so that they can perform tasks on these servers using Jenkins. Find below more details and accomplish the task accordingly.
> Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.
>
> 1. Add all app servers as SSH build agent/slave nodes in Jenkins. Slave node name for app server 1, app server 2 and app server 3 must be `App_server_1`, `App_server_2`, and `App_server_3` respectively.
> 2. Add labels as below:
>   * `App_server_1 : stapp01`
>   * `App_server_2 : stapp02`
>   * `App_server_3 : stapp03`
> 3. Remote root directory for App_server_1 must be `/home/tony/jenkins`, for App_server_2 must be `/home/steve/jenkins` and for App_server_3 must be `/home/banner/jenkins`.
> 4. Make sure slave nodes are online and working properly.
>
> **Note:** You might need to install some plugins and restart Jenkins service. So, we recommend clicking on Restart Jenkins when installation is complete and no jobs are running on plugin installation/update page i.e update centre. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case, please make sure to refresh the UI page. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Run sudo command via ssh
  * https://stackoverflow.com/a/10312700
* * Add SSH agent for builds via slaves.
  * https://plugins.jenkins.io/ssh-slaves/
* Add Jenkins SSH credential
  * https://www.jenkins.io/doc/book/using/using-credentials/
* Add a slave node.
  * https://www.baeldung.com/ops/jenkins-slave-node-setup


## Steps

```bash
# Install java
ssh -t tony@stapp01 'sudo dnf install -y java' # Ir0nM@n
ssh -t steve@stapp02 'sudo dnf install -y java'	# Am3ric@
ssh -t banner@stapp03 'sudo dnf install -y java' # BigGr33n
```

* Click the Jenkins button to access to UI through a broswer.
* Login with the credentials.
* Follow https://plugins.jenkins.io/ssh-slaves/ to install building on slaves via SSH.
* Follow https://www.jenkins.io/doc/book/using/using-credentials/#adding-new-global-credentials and store the SSH credentials.
* Follow https://www.baeldung.com/ops/jenkins-slave-node-setup to create the slave nodes listed above.

We are done.
