# Day 6 Task: Node.js and Python code deployment using Kubernetes-Minikube 

### In this project I have performed two tasks:
1. Deploying a Node.js App Using Minikube Kubernetes
2. Deploying a Python Flask App Using Minikube Kubernetes

### Project-1

First of all, I made a directory and initialized a Node.js project using below command:

```
npm init -y
```

This command will create a new package.json file in your Node.js project with default values. 

Then, I ran the below command

```
npm install express
```
Express is a minimal and flexible Node.js web application framework that provides a set of features for building your applications.

![alt text](images/Day_6_Images/Image_1)

Now, below is my index.js file:

```
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
    res.send('Hello, Kubernetes!');
});

app.listen(port, () => {
    console.log(`App running at http://localhost:${port}`);
});

app.get('/newroute', (req, res) => {
    res.send('This is a new route!');
});

```

All these tasks are done in one single directory.

This is my Dockerfile for the node code:

```
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

I built an image of this Dockefile and Pushed it to my Reposiory.

![alt text](images/Day_6_Images/Image_4)

When I ran the container out of this docker image, I was able to get my webpage as shown in below image:

![alt text](images/Day_6_Images/Image_3)

![alt text](images/Day_6_Images/Image_2)

These are my kubernetes menifest files which I used for the deployment:

deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - name: nodejs-app
        image: fansari9993/test9:tagname
        ports:
        - containerPort: 3000
```

service-nodeport.yaml:
```
apiVersion: v1
kind: Service
metadata:
  name: nodejs-service-nodeport
spec:
  selector:
    app: nodejs-app
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
    nodePort: 30001
  type: NodePort
```

service.yaml:

```
apiVersion: v1
kind: Service
metadata:
  name: nodejs-service
spec:
  selector:
    app: nodejs-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP
```

Afterwards, I deployed all three files using below commands:

```
kuectl apply -f deployment.yaml
kuectl apply -f service.yaml
kubectl apply -f service-nodeport.yaml
```

when I ran the command:

```
minikube service nodejs-service --url
```
I got the link on which I was able to get my desired web-page.

![alt text](images/Day_6_Images/Image_9)


# Deploying a Python Flask App Using Minikube Kubernetes

I made a directory in which I ran the command:

```
pip install flask
```
Flask is a Python-based web app framework that helps you develop lightweight and deployable web apps.

Below is my code:

```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Kubernetes!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

@app.route('/newroute')
def new_route():
    return 'This is a new route!'

```
I also created a requirements.txt file which contains a list of packages or libraries needed to work on a project that can all be installed with the file.


Below is my Dockerfile for the above code:

```
FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]

```

I used my Docker hub repository for pushing this image.

```
docker build -t fansari9993/test9:tagname .
docker push -t fansari9993/test9:tagname .
```

![text](images/Day_6_Images/Image_15)
![text](images/Day_6_Images/Image_16)
![text](images/Day_6_Images/Image_17)

Once, the file was pushed to the repository I tried to run a container using that image:

![text](images/Day_6_Images/Image_18)

```
docker run -p 5000:5000 fansari9993/test9:tagname
```
My container was up and running and giving me the desired wev-page as well.

Later on,
Below are my kubernets manifestation files which I used for the deployment purpose:

deployment.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
        - name: flask-app
          image: fansari9993/test9:tagname
          ports:
          - containerPort: 5000
```

service.yaml:
```
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  selector:
    app: flask-app
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
  type: ClusterIP
```

service-nodeport.yaml:
```
apiVersion: v1
kind: Service
metadata:
  name: flask-service-nodeport
spec:
  selector:
    app: flask-app
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
    nodePort: 30001
  type: NodePort
```

You can see that I was successfully I able to get my desired webpage:

![text](images/Day_6_Images/Image_21)




