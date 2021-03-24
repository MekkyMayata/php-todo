# Laravel Quickstart - Basic

## Quick Installation

    git clone https://github.com/laravel/quickstart-basic quickstart

    cd quickstart

    composer install

    php artisan migrate

    php artisan serve

[Complete Tutorial](https://laravel.com/docs/5.2/quickstart)

## CI Pipeline for TODO application
- The deployment of the application was carried out from Artifactory.

## STEPS
- Install Artifactory on the artifactory server
```
sudo apt-get install gnupg2 -y
wget -qO - https://api.bintray.com/orgs/jfrog/keys/gpg/public.key |sudo apt-key add -
sudo echo "deb https://jfrog.bintray.com/artifactory-debs bionic main" | sudo tee /etc/apt/sources.list.d/jfrog.list
sudo apt update -y
sudo apt install jfrog-artifactory-oss -y
sudo systemctl start artifactory
sudo systemctl enable artifactory
```
![project-14-artifactory-install-edit](https://user-images.githubusercontent.com/54307445/112258054-20d03180-8c66-11eb-9546-490d4596a18e.png)

- On the Jenkins server, Install PHP, its dependencies and Composer tool
```sudo yum install -y zip phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysqlnd,zip}```

- Install and configure Jenkins plugins (Plot, Artifactory)
![project-14-jfrog-artifactory-plugin](https://user-images.githubusercontent.com/54307445/112256061-bd450480-8c63-11eb-9dc8-7a49cea30859.png)

- In artifactory ui, create a repository that has the same name as your “target” in your jenkinsfile (under the Artifact deployment stage). In this case, it is php-todo
[Jenkinsfile](https://github.com/MekkyMayata/php-todo/blob/main/deploy/jenkinsfile)

- Create the necessary database requirements and edit the .env.sample file 

- Update the Jenkins file to include Unit tests step
[Jenkinsfile](https://github.com/MekkyMayata/php-todo/blob/main/deploy/jenkinsfile)

Next, setup code coverage report on the jenkins server.
 
1. run ```php --ini``` to see loaded modules and php config files.

2. from 1, try to find the xdebug file among the outputs, if its not there an install should be made (sudo yum/apt install php-xdebug)

3. run ```php --ini``` again and this time the xdebug file should be among the list of files.

4. edit the xdebug file in the output (run php --ini | grep xdebug) to locate the file. 

To edit, find the commented line (;xdebug.mode = develop) and change it to (xdebug.mode = coverage). 
The file also says something about passing a global environment variable XDEBUG_MODE to bypass the default one set in the file, so you may use this method also.

5. restart php (sudo systemctl restart php-fpm)

- run the job
![project-14-execute-unit-tests-edit](https://user-images.githubusercontent.com/54307445/112258316-989e5c00-8c66-11eb-9fa2-f47a9f76625b.png)

- add the code quality analytics stage to the pipeline and run it
![phploc-output](https://user-images.githubusercontent.com/54307445/112257487-3729bd80-8c65-11eb-9016-42c166738f97.png)

- Package the artifact and deploy to artifactory server
- Next, deploy to dev environment and configure sonarqube server (see [sonarqube role](https://github.com/MekkyMayata/CI-JAASP/tree/master/roles/sonarqube))
- Install the sonar plugin on Jenkins UI. Configure sonarqube and Jenkins for quality gates
![sonarqube-servers](https://user-images.githubusercontent.com/54307445/112258807-6b05e280-8c67-11eb-8fdb-d0205630b9b1.png)
![sonar-scanner](https://user-images.githubusercontent.com/54307445/112258715-40b42500-8c67-11eb-99b5-b55a0e78c8e5.png)

- edit the sonar.properties file located in ```/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf/```
```
#sonar.sourceEncoding=UTF-8

sonar.host.url=http://3.135.221.18:9000 # replace url here
sonar.projectKey=php-todo
#----- Default source code encoding
sonar.sourceEncoding=UTF-8
sonar.php.exclusions=**/vendor/**
sonar.php.coverage.reportPaths=build/logs/clover.xml
sonar.php.tests.reportPath=build/logs/junit.xml
```
- Update the jenkins pipeline to include sonarqube scanning stage
![project-14-sonar-report](https://user-images.githubusercontent.com/54307445/112259189-20389a80-8c68-11eb-81e6-d1209cbc2f7d.png)

- Add conditional deployment to higher environment stages in the jenkins pipeline
![project-14-sonar-bypass](https://user-images.githubusercontent.com/54307445/112259306-5b3ace00-8c68-11eb-978c-85b6447ad53a.png)

- Add 2 more servers to be used as Jenkins slave. Configure Jenkins to run its pipeline jobs randomly on any available slave nodes.
1. Under the managed jenkins, add nodes. In the node configuration section, provide necessary details. 
2. Also provide the private key of the node (jenkins will use it to connect). Make sure the java-jdk is installed on the slave(s).
![adding-nodes](https://user-images.githubusercontent.com/54307445/112259597-dbf9ca00-8c68-11eb-8128-4791506ce727.png)
![master-slave](https://user-images.githubusercontent.com/54307445/112259441-9b9a4c00-8c68-11eb-8472-b1608fa3ac74.png)

- Configure webhook between Jenkins and Github to automatically run the pipeline when there is a code push.
1. Install multibranch scan webhook trigger in manage plugins
2. Edit the scan repository triggers section of the multibranch pipeline. Also check the info and copy the syntax for the webhook( JENKINS_URL/multibranch-webhook-trigger/invoke?token=[Trigger token]). Replace accordingly (e.g; http://x.x.x.x:8080/multibranch-webhook-trigger/invoke?token=ci-jaasp-token-value)

![github-webhook](https://user-images.githubusercontent.com/54307445/112259775-21b69280-8c69-11eb-8944-e254b00f8cad.png)

3. Deploy application to remaining environments

## Inspired By:
- [Darey.io](https://darey.io)

🤗