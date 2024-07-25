# Task:12 Jenkins Pipeline 2

### First of all, I created a private git repo in which I pushed the code provided by the instructor. Then I created a pipeline project in Jenkins.

### I configured the Git private repo in Jenkins as below in which I kept branch specifier as main and Script path as Jenkinsfile.

![alt text](images/Day_12_Images/Image_3)

### Below is my Jenkinsfile which is used in my private git repo:

```
pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'Maven-3.9.0' // Ensure this matches your Maven tool name
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from GitHub repository
                git url: 'https://github.com/faraz9993/Private_Repo.git', branch: 'main', credentialsId: 'ca1559d6-3890-4fcb-9ead-11716016209a'
            }
        }

        stage('Build') {
            steps {
                // Build the project using Maven
                script {
                    // Debug step to print MAVEN_HOME
                    echo "MAVEN_HOME: ${MAVEN_HOME}"
                    withEnv(["PATH+MAVEN=${MAVEN_HOME}/bin"]) {
                        sh 'echo $PATH'  // Debug step to print PATH
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                // Archive the built artifacts
                archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
```

### When I build the pipeline it was successfully build and gave the stage output as shown in the below image.

![alt text](images/Day_12_Images/Image_2)

### Not only this, but all the artifacts were also appearing in the container as shown in the below image:


![alt text](images/Day_12_Images/Image_4)



