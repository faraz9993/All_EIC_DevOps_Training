### Day 14: In this task, I have done a set up of a CI/CD pipeline using Jenkins to streamline the deployment process of a simple Java application. 

### First of all, I created a simple application in Java as App.java which is given below:

```
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```
### I tried running this app in local first using below commands, which was running succesffully.
```
javac App.java
java App.java
```

### Secondly, I created a Dockerfile which which will create an image for this app.

```
FROM openjdk:11-slim
WORKDIR /app
COPY . /app
RUN javac App.java
CMD ["java", "App"]
```

### Next, I created a Jenkinsfile which uses my GitHub repo to build the image from the Dockerfile, will push that docker image to my docker repository and make a comtainer out of that image.

```
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        registry = 'docker.io'  
        registryCredential = 'personal-docker'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/faraz9993/Day_14.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t fansari9993/test9:latest .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'dockerhub-credentials-id', url: 'https://index.docker.io/v1/']) {
                        sh 'docker push fansari9993/test9:latest'
                    }
                }
            }
        }
        stage('Deploy Container') {
            steps {
                script {
                    sh 'docker run -d -p 8099:80 fansari9993/test9:latest'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
```
![alt text](images/Day_14_Images/Image_1)

### This configuration made a successful Jenkins pipeline.

![alt text](images/Day_14_Images/Image_2)

![alt text](images/Day_14_Images/Image_3)

