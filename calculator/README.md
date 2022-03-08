## Jenkins Exercises
The following exercises and source code (Java + Gradle repository) to learn Jenkins have been provided by Lemoncode here https://github.com/Lemoncode/bootcamp-devops-lemoncode/tree/master/03-cd/exercises 

1. CI/CD with Jenkins pipeline, Java + Gradle
Create a `Jenkinsfile` with a declarative pipeline declarativa with the following stages:
 
* **Checkout** download the source code from a remote repository, preferably GitHub.
* **Compile** compile the source code by using `gradlew compileJava`
* **Unit Tests** run the unit testsby using `gradlew test`
 
To run locally Jenkins and have the necessary dependencies availables we can build an image from [this Dockerfile](gradle.Dockerfile)
 
2. Modify the pipeline to use the Gradle Docker image as build runner
  In order to execute Docker commands inside Jenkins nodes, download and run the docker:dind Docker image.
  Docker and Docker Pipeline plugins must be installed
  Use the Docker image gradle:6.6.1-jre14-openj9
## Solution structure 


```
├── exercise1 (new)
│   ├── Jenkinsfile (new)
├── exercise2 (new)
│   ├── Jenkinsfile (new)
├── gradle
│   ├── wrapper
│   	├── gradle-wrapper.jar
│   	├── gradle-wrapper.properties
├── src
├── build.gradle
├── Dockerfile (gradle.Dockerfile)
├── gradlew
├── gradlew.bat
├── README.md
├── settings.gradle
├── start_jenkins.sh (new)
```

## Jenkins solution exercise1

Browse the classic UI http://localhost:8080

1. New item, pipeline, exercise1
2. Select pipeline from source control
3. Git - https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git
4. Ensure that the source control branch is main
5. Path to Jenkinsfile `calculator/exercise1/Jenkinsfile`
6. Build Now
7. Check
```
Started by user lemoncode
Obtained 03-cicd-jenkins/jenkins-resources/exercise1/Jenkinsfile from git https://github.com/monicacrespo/bootcamp-devops-student.git
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
 > git config remote.origin.url https://github.com/monicacrespo/bootcamp-devops-student.git # timeout=10
Fetching upstream changes from https://github.com/monicacrespo/bootcamp-devops-student.git
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git fetch --tags --force --progress -- https://github.com/monicacrespo/bootcamp-devops-student.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
Checking out Revision dce1088c0b978370f0aed6bbe41bca3eb11c58c6 (refs/remotes/origin/main)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f dce1088c0b978370f0aed6bbe41bca3eb11c58c6 # timeout=10
Commit message: "Created exercise1 folder"
 > git rev-list --no-walk 01aa2570f4718e2f9f45b436e94cf92b105f1a8a # timeout=10
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
 > git config remote.origin.url https://github.com/monicacrespo/bootcamp-devops-student.git # timeout=10
Fetching upstream changes from https://github.com/monicacrespo/bootcamp-devops-student.git
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git fetch --tags --force --progress -- https://github.com/monicacrespo/bootcamp-devops-student.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
Checking out Revision dce1088c0b978370f0aed6bbe41bca3eb11c58c6 (refs/remotes/origin/main)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f dce1088c0b978370f0aed6bbe41bca3eb11c58c6 # timeout=10
Commit message: "Created exercise1 folder"
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
Running in /var/jenkins_home/workspace/exercise1/03-cicd-jenkins/jenkins-resources/calculator
[Pipeline] {
[Pipeline] echo
Compiling project
[Pipeline] sh
+ chmod +x gradlew
+ ./gradlew compileJava
Starting a Gradle Daemon (subsequent builds will be faster)
> Task :compileJava

BUILD SUCCESSFUL in 13s
1 actionable task: 1 executed
[Pipeline] }
[Pipeline] // dir
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Unit Tests)
[Pipeline] dir
Running in /var/jenkins_home/workspace/exercise1/03-cicd-jenkins/jenkins-resources/calculator
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

BUILD SUCCESSFUL in 31s
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

For the Checkout stage I have looked at https://plugins.jenkins.io/git-parameter/

## Jenkins solution exercise2

Browse the classic UI http://localhost:8080

1. New item, pipeline, exercise2
2. Select pipeline from source control
3. Git - https://github.com/monicacrespo/bootcamp-devops-jenkins.git
4. Ensure that the source control branch is main
5. Path to Jenkinsfile `03-cicd-jenkins/jenkins-resources/exercise2/Jenkinsfile`
6. Build Now
7. Check

For using Docker with Pipeline I have looked at https://www.jenkins.io/doc/book/pipeline/docker/


```
agent {  
    dockerfile {
      dir './03-cicd-jenkins/jenkins-resources'    
      filename 'gradle.Dockerfile'
    }
  }
```

```
agent {
    docker {     
      image 'gradle'
    }
  }
```


## Jenkinsfile Validation
This is what I have used to validate the Jenkinsfile

```
cd 03-cicd-jenkins/jenkins-resources
cat exercise1/Jenkinsfile | curl --user lemoncode -X POST -F "jenkinsfile=<-" http://localhost:8080/pipeline-model-converter/validate
```

```
cat calculator/exercise2/Jenkinsfile | curl --user lemoncode -X POST -F "jenkinsfile=<-" http://localhost:8080/pipeline-model-converter/validate
```