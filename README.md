# jenkins CI/CD

## Installation of jenkins on aws 



* We have configured EC2 instance for our project 

![Alt text](<Screenshot from 2023-11-27 23-02-32.png>)

* Here we are installing jenkins through by pulling docker image

```bash
docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11
```
* Make sure to open the port 8080 in the inbound security rules

```txt
http://ipaddress:8080
```
* After sucessfull installtion of jenkins you can get the inital password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
* Then we need to configure our login credentials

![Alt text](<Screenshot from 2023-11-26 20-08-36.png>)

* we can see our landing page of jenkins 

![Alt text](<Screenshot from 2023-11-26 20-09-18.png>)

* Now we need to configure a job to deploy our python application

* Click on new item to create a project and name the project.

* ![Alt text](<Screenshot from 2023-11-27 23-10-52.png>)

* Give the git repo url

![Alt text](<Screenshot from 2023-11-27 23-13-10.png>)

* In the advance options there is pipeline script where we need to configure our pipelinescript to build the pipeline

```bash
pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *') // Poll SCM every 5 minutes
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CleanCheckout'], [$class: 'CloneOption', depth: 0, noTags: false, reference: '', shallow: true]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'harsha378', url: 'https://github.com/harsha378/sample.git']]])
            }
        }
      stage('Build') {
            steps {
                git branch: 'main', url: 'https://github.com/harsha378/sample.git'
                sh 'python3 ops.py'
            }
        }    
        stage('Test') {
            steps {
                sh 'python3 -m pytest'
            }
        }      
    }
}
post {
        success {
            echo 'Deployment successful!'
            // Send success email notification
            emailext attachLog: true, body: 'Deployment successful!', subject: 'Jenkins Pipeline - Success', to: 'kotasrharsha387@gmail.com'
        }
        failure {
            echo 'Deployment failed!'
            // Send failure email notification
            emailext attachLog: true, body: 'Deployment failed. Please check Jenkins for details.', subject: 'Jenkins Pipeline - Failure', to: 'kotasrharsha387@gmail.com'
        }
    }
```

* Now click on apply and build now to build the project 

* Hence our python project is sucessfully deployed 

* For every 5min the trigger will scan the repo made changes in repo and build.

* After every sucess build a notification is sent to mail configure through email availble plugin and mail send to mail if it fails then send to mail again.

![Alt text](<Screenshot from 2023-11-27 23-10-29.png>)