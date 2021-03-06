## Running Jenkins in docker

### Environment
These exercises are written based on the following requirements:

•	Operating System: Windows 10

•	Windows Subsystem for Linux: WSL2, Ubuntu-20.04

•	Docker: 20.10.12, build e91ed57

### Dockerfile
To run locally Jenkins and have the necessary dependencies availables you can build an image with Gradle from this [Dockerfile](gradle.Dockerfile) detailed in [Lemoncode](https://github.com/Lemoncode/bootcamp-devops-lemoncode/blob/master/03-cd/exercises/jenkins-resources/gradle.Dockerfile). The only change I have made has been the addition of Docker.

```
# Reference install customise official Jenkins Docker image: https://www.jenkins.io/doc/book/installing/docker/#downloading-and-running-jenkins-in-docker
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
```


The start_jenkins.sh script enables you to automate all the steps and is provided by [Lemoncode](https://github.com/Lemoncode/bootcamp-devops-lemoncode/tree/master/03-cd/01-jenkins/00-instalando-jenkins).


![RunningJenkins](https://github.com/monicacrespo/bootcamp-devops-student-jenkins/blob/main/RunningJenkins.JPG)

### Steps in this order using WSL2 terminal
1. Ensure that script have permissions to be executed: `chmod +x start_jenkins.sh`
2. Build image from Dockerfile: `docker build -t lemoncode/jenkins .`
3. Run Jenkins in Docker. From previous image we can run custom Jenkins version as follows:
    `./start_jenkins.sh lemoncode/jenkins jenkins jenkins-docker-certs jenkins-data`
4. Proceed to the [Post-installation setup wizard](https://www.jenkins.io/doc/book/installing/docker/#setup-wizard)

 
## Jenkins Exercises
The following exercises and source code (Java + Gradle repository) to learn Jenkins have been provided by Lemoncode here https://github.com/Lemoncode/bootcamp-devops-lemoncode/tree/master/03-cd/exercises 

1. CI/CD with Jenkins pipeline, Java + Gradle. Create a `Jenkinsfile` with a declarative pipeline with the following stages:
 
* **Checkout** download the source code from a remote repository, preferably GitHub.
* **Compile** compile the source code by using `gradlew compileJava`
* **Unit Tests** run the unit testsby using `gradlew test`
 
2. Modify the pipeline to use the Gradle Docker image as build runner.
    * In order to execute Docker commands inside Jenkins nodes, download and run the docker:dind Docker image.
    * Docker and Docker Pipeline plugins must be installed
    * Use the Docker image gradle:6.6.1-jre14-openj9


## Solution structure 

```
├── calculator
│   ├── exercise1 (new)
│   	├── Jenkinsfile (new)
│   ├── exercise2 (new)
│   	├── Jenkinsfile (new)
│   ├── gradle
│   ├── wrapper
│   	├── gradle-wrapper.jar
│     	├── gradle-wrapper.properties
│   ├── src
│   ├── build.gradle
│   ├── gradlew
│   ├── gradlew.bat
│   ├── settings.gradle
├── gradle.Dockerfile (modified to include Docker)
├── README.md
├── start_jenkins.sh
```


## Jenkins solution exercise1

### Jenkinsfile script
You can also find it in the current [repository](https://github.com/monicacrespo/bootcamp-devops-student-jenkins/blob/main/calculator/exercise1/Jenkinsfile)
```
pipeline {
  agent any

  stages {
	
   stage('Checkout code') {
      steps {
        echo "Checking out project"
        // GIT checkout
        checkout scm: [
                $class: 'GitSCM',
                branches: [[name: '*/main']], 
                doGenerateSubmoduleConfigurations: false, 
                extensions: [[$class: 'CleanCheckout']], 
                submoduleCfg: [], 
                userRemoteConfigs: [[url: 'https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git']]
        ]
      }
    }
    stage('Compile') {
      steps {    
        dir('./calculator'){
            echo "Compiling project"
            sh '''
            chmod +x gradlew
            ./gradlew compileJava
            '''
        }   
      }
    }
    stage('Unit Tests') {
      steps {
        dir('./calculator'){
          echo "Running unit tests" 
          sh './gradlew test'
        }
      }
    }
  }
}

```
For the Checkout stage I have looked at https://plugins.jenkins.io/git-parameter/

### Defining a Pipeline through the classic UI
To create a basic Pipeline through the Jenkins classic UI:
1. Browse the classic UI http://localhost:8080
2. From the Jenkins home page (i.e. the Dashboard of the Jenkins classic UI), click New Item at the top left.
3. In the `Enter an item name` field, specify the name for your new Pipeline project, i.e.`exercise1`
4. Scroll down and click Pipeline, then click "OK" at the end of the page to open the Pipeline configuration page.
5. Click the Pipeline tab at the top of the page to scroll down to the Pipeline section.
6. Select pipeline from source control
7. From the Definition field, choose the Pipeline script from SCM option.
8. From the SCM field, choose the type of source control system of the repository containing your Jenkinsfile, i.e. "Git".
9. In the `Repository URL` field, enter `https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git`
10. In the `Branch Specifier (blank for 'any')` field, ensure that the source control branch is `*/main`
11. In the `Script Path` field, specify the location (and name) of your Jenkinsfile, i.e. `calculator/exercise1/Jenkinsfile`. This location is the one that Jenkins checks out/clones the repository containing your Jenkinsfile, which should match that of the repository’s file structure. 
12. Build Now
13. Check. Open console from running build

    Console Output
    ```
    Started by user lemoncode
    Obtained calculator/exercise1/Jenkinsfile from git https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git
    [Pipeline] Start of Pipeline
    [Pipeline] node
    Running on Jenkins in /var/jenkins_home/workspace/exercise1
    [Pipeline] {
    [Pipeline] stage
    [Pipeline] { (Declarative: Checkout SCM)
    [Pipeline] checkout
    Selected Git installation does not exist. Using Default
    The recommended git tool is: NONE
    No credentials specified
    > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/exercise1/.git # timeout=10
    Fetching changes from the remote Git repository
    > git config remote.origin.url https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git # timeout=10
    Fetching upstream changes from https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git
    > git --version # timeout=10
    > git --version # 'git version 2.30.2'
    > git fetch --tags --force --progress -- https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git +refs/heads/*:refs/remotes/origin/* # timeout=10
    > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
    Checking out Revision 38a0eeb9a9af48afda9f88c54e7787a8536a2601 (refs/remotes/origin/main)
    > git config core.sparsecheckout # timeout=10
    > git checkout -f 38a0eeb9a9af48afda9f88c54e7787a8536a2601 # timeout=10
    Commit message: "Modified README.md"
    > git rev-list --no-walk e891aca64a5a555f75288c83c1cd1e9740de8ed9 # timeout=10
    [Pipeline] }
    [Pipeline] // stage
    [Pipeline] withEnv
    [Pipeline] {
    [Pipeline] stage
    [Pipeline] { (Checkout code)
    [Pipeline] echo
    Checking out project
    [Pipeline] checkout
    Selected Git installation does not exist. Using Default
    The recommended git tool is: NONE
    No credentials specified
    > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/exercise1/.git # timeout=10
    Fetching changes from the remote Git repository
    > git config remote.origin.url https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git # timeout=10
    Fetching upstream changes from https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git
    > git --version # timeout=10
    > git --version # 'git version 2.30.2'
    > git fetch --tags --force --progress -- https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git +refs/heads/*:refs/remotes/origin/* # timeout=10
    > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
    Checking out Revision 38a0eeb9a9af48afda9f88c54e7787a8536a2601 (refs/remotes/origin/main)
    > git config core.sparsecheckout # timeout=10
    > git checkout -f 38a0eeb9a9af48afda9f88c54e7787a8536a2601 # timeout=10
    Commit message: "Modified README.md"
    Cleaning workspace
    > git rev-parse --verify HEAD # timeout=10
    Resetting working tree
    > git reset --hard # timeout=10
    > git clean -fdx # timeout=10
    [Pipeline] }
    [Pipeline] // stage
    [Pipeline] stage
    [Pipeline] { (Compile)
    [Pipeline] dir
    Running in /var/jenkins_home/workspace/exercise1/calculator
    [Pipeline] {
    [Pipeline] echo
    Compiling project
    [Pipeline] sh
    + chmod +x gradlew
    + ./gradlew compileJava
    Starting a Gradle Daemon, 1 busy Daemon could not be reused, use --status for details
    > Task :compileJava

    BUILD SUCCESSFUL in 15s
    1 actionable task: 1 executed
    [Pipeline] }
    [Pipeline] // dir
    [Pipeline] }
    [Pipeline] // stage
    [Pipeline] stage
    [Pipeline] { (Unit Tests)
    [Pipeline] dir
    Running in /var/jenkins_home/workspace/exercise1/calculator
    [Pipeline] {
    [Pipeline] echo
    Running unit tests
    [Pipeline] sh
    + ./gradlew test
    > Task :compileJava UP-TO-DATE
    > Task :processResources
    > Task :classes
    > Task :compileTestJava
    > Task :processTestResources NO-SOURCE
    > Task :testClasses
    > Task :test

    BUILD SUCCESSFUL in 7s
    4 actionable tasks: 3 executed, 1 up-to-date
    [Pipeline] }
    [Pipeline] // dir
    [Pipeline] }
    [Pipeline] // stage
    [Pipeline] }
    [Pipeline] // withEnv
    [Pipeline] }
    [Pipeline] // node
    [Pipeline] End of Pipeline
    Finished: SUCCESS
    ```

    ![Pipeline Exercise1](https://github.com/monicacrespo/bootcamp-devops-student-jenkins/blob/main/PipelineExercise1.JPG)


## Jenkins solution exercise2

### Jenkinsfile script
You can also find it in the current [repository](https://github.com/monicacrespo/bootcamp-devops-student-jenkins/blob/main/calculator/exercise2/Jenkinsfile)

Jenkinsfile script
```
pipeline {
  agent {
    docker {
      image 'gradle:6.6.1-jre14-openj9'
    }
  }

  stages {
    stage('Verify') {
      steps {   
        sh 'ls -l "$WORKSPACE"'                           
      }
    }
   stage('Checkout code') {
      steps {
        echo "Checking out project"
        // GIT checkout
        checkout scm: [
                $class: 'GitSCM',
                branches: [[name: '*/main']], 
                doGenerateSubmoduleConfigurations: false, 
                extensions: [[$class: 'CleanCheckout']], 
                submoduleCfg: [], 
                userRemoteConfigs: [[url: 'https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git']]
        ]
      }
    }
    stage('Compile') {
      steps {
        dir("$WORKSPACE/calculator"){
            echo "Compiling project"
            sh '''
            chmod +x gradlew
            ./gradlew compileJava
            '''
        }    
      }
    }
    stage('Unit Tests') {
      steps {
        dir("$WORKSPACE/calculator"){
          echo "Running unit tests" 
          sh './gradlew test'
        }
      }
    }
  } 
}
```
For using Docker with Pipeline I have looked at https://www.jenkins.io/doc/book/pipeline/docker/

Note that I have added an extra stage called Verify to list the value of $WORKSPACE value.
You could see in the console output below that the result of ls value of `sh 'ls -l "$WORKSPACE"'`  is the following:

```
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Verify)
[Pipeline] sh
+ ls -l /var/jenkins_home/workspace/exercise2
total 60
drwxr-xr-x 8 gradle gradle  4096 Mar 21 12:20 calculator
drwxr-xr-x 2 gradle gradle  4096 Mar 21 12:17 calculator@tmp
-rw-r--r-- 1 gradle gradle  1790 Mar 18 10:34 gradle.Dockerfile
-rw-r--r-- 1 gradle gradle  6686 Mar 21 12:20 README.md
-rw-r--r-- 1 gradle gradle 34746 Mar 18 10:34 RunningJenkins.JPG
-rw-r--r-- 1 gradle gradle  1661 Mar 18 10:34 start_jenkins.sh
[Pipeline] }
[Pipeline] // stage
```

### Defining a Pipeline through the classic UI
To create a Pipeline from other existing:
1. Browse the classic UI http://localhost:8080
2. From the Jenkins home page (i.e. the Dashboard of the Jenkins classic UI), click New Item at the top left.
3. In the `Enter an item name` field, specify the name for your new Pipeline project, i.e.`exercise2`
4. Scroll Downs and in the `Copy From` field, enter `exercise1` to create a new item from other existing. Then click "OK" at the end of the page to open the Pipeline configuration page.
5. In the `Script Path` field, specify the new location (and name) of your Jenkinsfile, i.e. `calculator/exercise2/Jenkinsfile`. 
6. Build Now
7. Check. Open console from running build

    Console Output
    ```
    Started by user lemoncode
    Obtained calculator/exercise2/Jenkinsfile from git https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git
    [Pipeline] Start of Pipeline
    [Pipeline] node
    Running on Jenkins in /var/jenkins_home/workspace/exercise2
    [Pipeline] {
    [Pipeline] stage
    [Pipeline] { (Declarative: Checkout SCM)
    [Pipeline] checkout
    Selected Git installation does not exist. Using Default
    The recommended git tool is: NONE
    No credentials specified
    > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/exercise2/.git # timeout=10
    Fetching changes from the remote Git repository
    > git config remote.origin.url https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git # timeout=10
    Fetching upstream changes from https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git
    > git --version # timeout=10
    > git --version # 'git version 2.30.2'
    > git fetch --tags --force --progress -- https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git +refs/heads/*:refs/remotes/origin/* # timeout=10
    > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
    Checking out Revision 4c354820d3e398fb6308c5b6751ed1e6913e1d12 (refs/remotes/origin/main)
    > git config core.sparsecheckout # timeout=10
    > git checkout -f 4c354820d3e398fb6308c5b6751ed1e6913e1d12 # timeout=10
    Commit message: "removed docker --version"
    > git rev-list --no-walk 8ba9003e95216777ff3bbc6291c366ace13f9e89 # timeout=10
    [Pipeline] }
    [Pipeline] // stage
    [Pipeline] withEnv
    [Pipeline] {
    [Pipeline] isUnix
    [Pipeline] withEnv
    [Pipeline] {
    [Pipeline] sh
    + docker inspect -f . gradle:6.6.1-jre14-openj9
    .
    [Pipeline] }
    [Pipeline] // withEnv
    [Pipeline] withDockerContainer
    Jenkins seems to be running inside container 562ea77cee41f3bd2e3f41e642278b244df0b0e4a67bef18c71ce350074d207f
    but /var/jenkins_home/workspace/exercise2 could not be found among []
    but /var/jenkins_home/workspace/exercise2@tmp could not be found among []
    $ docker run -t -d -u 1000:1000 -w /var/jenkins_home/workspace/exercise2 -v /var/jenkins_home/workspace/exercise2:/var/jenkins_home/workspace/exercise2:rw,z -v /var/jenkins_home/workspace/exercise2@tmp:/var/jenkins_home/workspace/exercise2@tmp:rw,z -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** gradle:6.6.1-jre14-openj9 cat
    $ docker top 1b7744d7991ab2b8c96cbe6a1b8052fb2db967ff3bacad0673feecb8f95af8d6 -eo pid,comm
    [Pipeline] {
    [Pipeline] stage
    [Pipeline] { (Verify)
    [Pipeline] sh
    + ls -l /var/jenkins_home/workspace/exercise2
    total 60
    drwxr-xr-x 8 gradle gradle  4096 Mar 21 12:20 calculator
    drwxr-xr-x 2 gradle gradle  4096 Mar 21 12:17 calculator@tmp
    -rw-r--r-- 1 gradle gradle  1790 Mar 18 10:34 gradle.Dockerfile
    -rw-r--r-- 1 gradle gradle  6686 Mar 21 12:20 README.md
    -rw-r--r-- 1 gradle gradle 34746 Mar 18 10:34 RunningJenkins.JPG
    -rw-r--r-- 1 gradle gradle  1661 Mar 18 10:34 start_jenkins.sh
    [Pipeline] }
    [Pipeline] // stage
    [Pipeline] stage
    [Pipeline] { (Checkout code)
    [Pipeline] echo
    Checking out project
    [Pipeline] checkout
    Selected Git installation does not exist. Using Default
    The recommended git tool is: NONE
    No credentials specified
    Warning: JENKINS-30600: special launcher org.jenkinsci.plugins.docker.workflow.WithContainerStep$Decorator$1@2db03124; decorates hudson.Launcher$LocalLauncher@5ad4ca4a will be ignored (a typical symptom is the Git executable not being run inside a designated container)
    > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/exercise2/.git # timeout=10
    Fetching changes from the remote Git repository
    > git config remote.origin.url https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git # timeout=10
    Fetching upstream changes from https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git
    > git --version # timeout=10
    > git --version # 'git version 2.30.2'
    > git fetch --tags --force --progress -- https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git +refs/heads/*:refs/remotes/origin/* # timeout=10
    > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
    Checking out Revision 4c354820d3e398fb6308c5b6751ed1e6913e1d12 (refs/remotes/origin/main)
    > git config core.sparsecheckout # timeout=10
    > git checkout -f 4c354820d3e398fb6308c5b6751ed1e6913e1d12 # timeout=10
    Commit message: "removed docker --version"
    Cleaning workspace
    > git rev-parse --verify HEAD # timeout=10
    Resetting working tree
    > git reset --hard # timeout=10
    > git clean -fdx # timeout=10
    [Pipeline] }
    [Pipeline] // stage
    [Pipeline] stage
    [Pipeline] { (Compile)
    [Pipeline] dir
    Running in /var/jenkins_home/workspace/exercise2/calculator
    [Pipeline] {
    [Pipeline] echo
    Compiling project
    [Pipeline] sh
    + chmod +x gradlew
    + ./gradlew compileJava
    Downloading https://services.gradle.org/distributions/gradle-6.6.1-bin.zip
    .........10%..........20%..........30%..........40%.........50%..........60%..........70%..........80%..........90%.........100%

    Welcome to Gradle 6.6.1!

    Here are the highlights of this release:
    - Experimental build configuration caching
    - Built-in conventions for handling credentials
    - Java compilation supports --release flag

    For more details see https://docs.gradle.org/6.6.1/release-notes.html

    Starting a Gradle Daemon (subsequent builds will be faster)
    > Task :compileJava

    BUILD SUCCESSFUL in 2m 35s
    1 actionable task: 1 executed
    [Pipeline] }
    [Pipeline] // dir
    [Pipeline] }
    [Pipeline] // stage
    [Pipeline] stage
    [Pipeline] { (Unit Tests)
    [Pipeline] dir
    Running in /var/jenkins_home/workspace/exercise2/calculator
    [Pipeline] {
    [Pipeline] echo
    Running unit tests
    [Pipeline] sh
    + ./gradlew test
    > Task :compileJava UP-TO-DATE
    > Task :processResources
    > Task :classes
    > Task :compileTestJava
    > Task :processTestResources NO-SOURCE
    > Task :testClasses
    > Task :test

    BUILD SUCCESSFUL in 34s
    4 actionable tasks: 3 executed, 1 up-to-date
    [Pipeline] }
    [Pipeline] // dir
    [Pipeline] }
    [Pipeline] // stage
    [Pipeline] }
    $ docker stop --time=1 1b7744d7991ab2b8c96cbe6a1b8052fb2db967ff3bacad0673feecb8f95af8d6
    $ docker rm -f 1b7744d7991ab2b8c96cbe6a1b8052fb2db967ff3bacad0673feecb8f95af8d6
    [Pipeline] // withDockerContainer
    [Pipeline] }
    [Pipeline] // withEnv
    [Pipeline] }
    [Pipeline] // node
    [Pipeline] End of Pipeline
    Finished: SUCCESS
    ```

    ![Pipeline Exercise2](https://github.com/monicacrespo/bootcamp-devops-student-jenkins/blob/main/PipelineExercise2.JPG)


Pipeline Exercise 1 and Exercise 2

![Pipeline Exercises 1 And 2](https://github.com/monicacrespo/bootcamp-devops-student-jenkins/blob/main/PipelineExercises1And2.JPG)


## Jenkinsfile Validation
This is the command I have used to validate the Jenkinsfile

```
cat calculator/exercise1/Jenkinsfile | curl --user lemoncode -X POST -F "jenkinsfile=<-" http://localhost:8080/pipeline-model-converter/validate
```

```
cat calculator/exercise2/Jenkinsfile | curl --user lemoncode -X POST -F "jenkinsfile=<-" http://localhost:8080/pipeline-model-converter/validate
```