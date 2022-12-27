# jenkins-maven-project

Java and Maven Jobs in Jenkins

Purpose of the this hands-on training is to learn how to install Java and Maven to Jenkins Server and configure Maven/Java Jobs.

## Learning Outcomes

At the end of the this hands-on training, students will be able to;

- install and configure Maven,

- create Java and Maven jobs

- create job DSL


## Outline

- Part 1 - Install Java, Maven and Git packages

- Part 2 - Maven Settings

- Part 3 - Creating Package Application - Free Style Maven Job (in java-tomcat-sample repo) 

- Part 4 - Configuring Jenkins Pipeline with GitHub Webhook to Build the Java Code (in  jenkinsfile-pipeline-project repo)

- Part 5 - Configuring Jenkins Pipeline with GitHub Webhook to Build the a Java Maven Project

- Part 6 - Jenkins Job DSL

## Part 1 - Install Java, Maven and Git packages

- Connect to the Jenkins Server 
  
- Install Java
  
```bash
sudo yum update -y
sudo amazon-linux-extras install java-openjdk11 -y
sudo yum install java-devel 
```

- Install Maven
  
```bash
sudo su
cd /opt
rm -rf maven
wget https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz
tar -zxvf $(ls | grep apache-maven-*-bin.tar.gz)
rm -rf $(ls | grep apache-maven-*-bin.tar.gz)
sudo ln -s $(ls | grep apache-maven*) maven
echo 'export M2_HOME=/opt/maven' > /etc/profile.d/maven.sh
echo 'export PATH=${M2_HOME}/bin:${PATH}' >> /etc/profile.d/maven.sh
exit
source /etc/profile.d/maven.sh
```
- Install Git
  
```bash
sudo yum install git -y
```

## Part 2 - Maven Settings

- Open Jenkins GUI on web browser

- Setting System Maven Path for default usage
  
- Go to `Manage Jenkins`
  - Select `Configure System`
  - Find `Environment variables` part,
  - Click `Add`
    - for `Name`, enter `PATH+EXTRA` 
    - for `Value`, enter `/opt/maven/bin`
- Save

- Setting a specific Maven Release in Jenkins for usage
  
- Go to the `Global Tool Configuration`
- To the bottom, `Maven` section
  - Give a name such as `maven-3.8.5`
  - Select `install automatically`
  - `Install from Apache` version `3.8.5`
- Save


## Part 5 - Configuring Jenkins Pipeline with GitHub Webhook to Build the a Java Maven Project

- To build the `java maven project` with Jenkins pipeline using the `Jenkinsfile` and `GitHub Webhook`. To accomplish this task, we need;

  - a java code to build

  - a java environment to run the build stages on the java code

  - a maven environment to run the build stages on the java code

  - a Jenkinsfile configured for an automated build on our repo

- Create a public project repository `jenkins-maven-project` on your GitHub account.

- Clone the `jenkins-maven-project` repository on local computer.

- Copy the files given within the hands-on folder [hello-app](./hello-app)  and paste under the `jenkins-maven-project` GitHub repo folder.

- Go to your Github `jenkins-maven-project` repository page and click on `Settings`.

- Click on the `Webhooks` on the left hand menu, and then click on `Add webhook`.

- Copy the Jenkins URL from the AWS Management Console, paste it into `Payload URL` field, add `/github-webhook/` at the end of URL, and click on `Add webhook`.

```text
http://ec2-54-144-151-76.compute-1.amazonaws.com:8080/github-webhook/
```

- Go to the Jenkins dashboard and click on `New Item` to create a pipeline.

- Enter `pipeline-with-jenkinsfile-and-webhook-for-maven-project` then select `Pipeline` and click `OK`.

- Enter `Simple pipeline configured with Jenkinsfile and GitHub Webhook for Maven project` in the description field.

- Put a checkmark on `GitHub Project` under `General` section, enter URL of the project repository.

```text
https://github.com/<your_github_account_name>/jenkins-maven-project/
```

- Put a checkmark on `GitHub hook trigger for GITScm polling` under `Build Triggers` section.

- Go to the `Pipeline` section, and select `Pipeline script from SCM` in the `Definition` field.

- Select `Git` in the `SCM` field.

- Enter URL of the project repository, and let others be default.

```text
https://github.com/<your_github_account_name>/jenkins-maven-project/
```

- Click `apply` and `save`. Note that the script `Jenkinsfile` should be placed under root folder of repo.

- Create a `Jenkinsfile` with the following pipeline script, and explain the script.

- For native structured Jenkins Server

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn -f hello-app/pom.xml -B -DskipTests clean package'
            }
            post {
                success {
                    echo "Now Archiving the Artifacts....."
                    archiveArtifacts artifacts: '**/*.jar'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn -f hello-app/pom.xml test'
            }
            post {
                always {
                    junit 'hello-app/target/surefire-reports/*.xml'
                }
            }
        }
    }
}
```

- Commit and push the changes to the remote repo on GitHub.

```bash
git add .
git commit -m 'added jenkinsfile and maven project'
git push
```

- Observe the new built triggered with `git push` command on the Jenkins project page.

- Explain the role of the docker image of maven, `Jenkinsfile` and GitHub Webhook in this automation.

## Part 6 - Jenkins Job DSL

- Install `Job DSL` plugin

- Go to the dashboard and create a `Seed Job` in form of `Free Style Project`. To do so;

- Click on `New Item`

  - Enter name as `Maven-Seed-Job`

  - Select `Freestyle project`

  - Click `OK`

- Inside `Source Code Management` tab

  - Select `Git`
  
  - Select the path to download the DSL file, so for `Repository URL`, enter `https://github.com/JBCodeWorld/jenkins-project-settings.git`

- Inside `Build Options` tab

  - From `Add build step`, select the `Process Job DSLs`.

  - for `Look on Filesystem` `DSL Scripts`, enter `MavenProjectDSL.groovy`
  
- Now click the  `Build Now` option, it will fail. Check the console log for the fail reason.

- Go to `Manage Jenkins` ,  select the `In-process Script Approval`, `approve` the script.
  
- Go to the job and click the  `Build Now` again.

- Observe that DSL Job created.

- Go to the `First-Maven-Project-Via-DSL` job.

- Select `Configure`, at `Build` section set `Maven Version` to a defined/valid one.

- `Save` and click the `Build Now` option.

- Check the console log

- Note: git configuration needs to be set for Jenkins user
    
    - Switch to "jenkins" user and configure git.

```bash
sudo su - jenkins
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

- Back to the job tab and show the `Last Successful Artifacts : single-module-project.jar`
