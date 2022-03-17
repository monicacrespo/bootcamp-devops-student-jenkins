## Jenkins Exercises
The following exercises and source code (Java + Gradle repository) to learn Jenkins have been provided by Lemoncode here https://github.com/Lemoncode/bootcamp-devops-lemoncode/tree/master/03-cd/exercises 

1. CI/CD with Jenkins pipeline, Java + Gradle. Create a `Jenkinsfile` with a declarative pipeline with the following stages:
 
* **Checkout** download the source code from a remote repository, preferably GitHub.
* **Compile** compile the source code by using `gradlew compileJava`
* **Unit Tests** run the unit testsby using `gradlew test`
 
To run locally Jenkins and have the necessary dependencies availables you can build an image from [this Dockerfile](gradle.Dockerfile)
 
2. Modify the pipeline to use the Gradle Docker image as build runner.
    * In order to execute Docker commands inside Jenkins nodes, download and run the docker:dind Docker image.
    * Docker and Docker Pipeline plugins must be installed
    * Use the Docker image gradle:6.6.1-jre14-openj9


## Solution structure 

```
├── calculator
├── ├──exercise1 (new)
│     ├── Jenkinsfile (new)
├── ├──exercise2 (new)
│     ├── Jenkinsfile (new)
├── ├──gradle
│     ├── wrapper
│   	  ├── gradle-wrapper.jar
│     	├── gradle-wrapper.properties
├── ├──src
├── ├──build.gradle
├── ├──gradlew
├── ├──gradlew.bat
├── ├──settings.gradle
├── gradle.Dockerfile (modified to include Docker)
├── README.md
├── start_jenkins.sh
```

## Jenkins solution exercise1

Browse the classic UI http://localhost:8080

1. New item, pipeline, `exercise1`
2. Select pipeline from source control
3. Git - https://github.com/monicacrespo/bootcamp-devops-student-jenkins.git
4. Ensure that the source control branch is main
5. Path to Jenkinsfile `calculator/exercise1/Jenkinsfile`
6. Build Now
7. Check

Jenkinsfile
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


## Jenkins solution exercise2

Browse the classic UI http://localhost:8080

1. New item, pipeline, exercise2
2. Select pipeline from source control
3. Git - https://github.com/monicacrespo/bootcamp-devops-jenkins.git
4. Ensure that the source control branch is main
5. Path to Jenkinsfile `calculator/exercise2/Jenkinsfile`
6. Build Now
7. Check

Jenkinsfile
```
pipeline {
  agent {
    docker {
      image 'gradle'
    }
  }

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

For using Docker with Pipeline I have looked at https://www.jenkins.io/doc/book/pipeline/docker/

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
cat calculator/exercise1/Jenkinsfile | curl --user lemoncode -X POST -F "jenkinsfile=<-" http://localhost:8080/pipeline-model-converter/validate
```

```
cat calculator/exercise2/Jenkinsfile | curl --user lemoncode -X POST -F "jenkinsfile=<-" http://localhost:8080/pipeline-model-converter/validate
```