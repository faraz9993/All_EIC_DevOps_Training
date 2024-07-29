### Day:15 In this task, I have used Github, Docker, Jenkins CI/CD and ansible to deploy a docker container on the target machine.

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

### Secondly, I created a Dockerfile which will create an image for this app.

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

### Later on, I created a virtual machine and got its IP which is 192.168.232.211. I used a that IP and created an inventory file and ping it.

```
[webservers]
192.168.232.211 ansible_user=fansari ansible_ssh_pass=fansari1234 ansible_become_pass=fansari1234
```

### I ping my target machine using below command:
```
ansible all -i inventory.cfg -m ping
```

![alt text](images/Day_15_Images/Image_1)

### Furthermore, I created a YAML file to execute the needed tasks on the target machine. Below is my YAML file.


```
---
- name: Deploying container
  hosts: webservers
  become: true
  tasks:
    - name: Instaling python pip
      apt:
        name: python3-pip
        state: present
        update_cache: yes

    - name: Installing Docker SDK
      pip:
        name: docker
        state: present

    - name: Installing Docker
      apt:
        name: docker.io
        state: present
        update_cache: yes

    - name: Making dokcer service running
      service:
        name: docker
        state: started
        enabled: yes

    - name: Pulling docker image
      docker_image:
        name: fansari9993/test9
        tag: latest
        source: pull

    - name: Running docker container
      docker_container:
        name: my_test_container
        image: fansari9993/test9:latest
        state: started
        restart_policy: always
        ports:
          - "8091:80"
```

### I ran this YAML file using below command:
```
ansible-playbook -i inventory.cfg main.yaml
```
![alt text](images/Day_15_Images/Image_2)

### I was able to see my docker container running on the target machine as it can be seen in the below image:

![alt text](images/Day_15_Images/Image_3)



