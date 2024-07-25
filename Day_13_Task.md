# Day 13: Multi-Branch Pipeline for a Simple Java Maven Project

### First of all, I created a git repo named my-java-app. In that repo, I created below file structure:

```
my-java-app/
├── pom.xml
└── src
    └── main
        └── java
            └── com
                └── example
                    └── App.java
```

### The content inside my pom.xml file is:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/POM/4.0.0/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-java-app</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
</project>
```
### The content inside my App.java is:

```
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Hello, Maven!");
    }
}
```


### Now, I created two more branches named feature-branch-1 & feature-branch-2.

### As I created two more branches, all the files available in the master branch wento to other branches as well.

### I modified the content of App.java in feature-branch-1 as:

```
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Feature Branch 1!");
    }
}
```

### I modified the content of App.java in feature-branch-2 as:

```
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Feature Branch 2!");
    }
}
```

### Then, I created a Jenkinsfile in master branch:

```
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/faraz9993/my-java-app.git', branch: env.BRANCH_NAME
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building production branch: ${env.BRANCH_NAME}"
                    withMaven(maven: 'Maven-3.9.0') {
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Run') {
            steps {
                script {
                    echo "Running Java application"
                    sh 'java -cp target/my-java-app-1.0-SNAPSHOT.jar com.example.App'
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                script {
                    echo "Deploying to production from branch: ${env.BRANCH_NAME}"
                }
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

### The Jenkinsfile of feature-branch-1 is:
```
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/faraz9993/my-java-app.git', branch: env.BRANCH_NAME
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building production branch: ${env.BRANCH_NAME}"
                    withMaven(maven: 'Maven-3.9.0') {
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Run') {
            steps {
                script {
                    echo "Running Java application"
                    sh 'java -cp target/my-java-app-1.0-SNAPSHOT.jar com.example.App'
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'feature-branch-1'
            }
            steps {
                script {
                    echo "Deploying to production from branch: ${env.BRANCH_NAME}"
                }
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

### The Jenkinsfile of feature-branch-2 is:

```
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/faraz9993/my-java-app.git', branch: env.BRANCH_NAME
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building production branch: ${env.BRANCH_NAME}"
                    withMaven(maven: 'Maven-3.9.0') {
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Run') {
            steps {
                script {
                    echo "Running Java application"
                    sh 'java -cp target/my-java-app-1.0-SNAPSHOT.jar com.example.App'
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'feature-branch-2'
            }
            steps {
                script {
                    echo "Deploying to production from branch: ${env.BRANCH_NAME}"
                }
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

### In all three branches, the environment variables are changed as per the branches names.

### Next, I created a Multi-branch Pipeline job in Jenkins and configured it with my GitHub Repo.

![alt text](images/Day_13_Images/Image_2)

### When I build, I was able to get my desired output in all three branches as shown in below images:

![alt text](images/Day_13_Images/Image_6)

### For master branch:
![alt text](images/Day_13_Images/Image_3)

### For feature-branch-1:
![alt text](images/Day_13_Images/Image_4)

### For feature-branch-2:
![alt text](images/Day_13_Images/Image_5)

