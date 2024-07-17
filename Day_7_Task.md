# Day 7 Task: Node.js code deployment using minikube-kubernetes with ConfigMaps, Secrets, environment variables and horizontal pod autoscaling

### In this task I have developed a simple Node.js application, deploy it on a local Kubernetes cluster using Minikube, and configure various Kubernetes features.

First of all, I created a directory named "nodejs-k8s-project" where I have to performed all the task.

First of all I ran the command

```
npm init -y
```
This command will create a new package.json file in your Node.js project directory with default values. 

Then, I ran the below command

```
npm install express body-parser
```

express body-parser is an npm module used to process data sent in an HTTP request body.

![alt text](images/Day_7_Images/Image_2)

This is my app.js file:

```
const express = require('express');
const bodyParser = require('body-parser');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(bodyParser.json());

app.get('/', (req, res) => {
  res.send('Hello, World!');
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```
Below is my Dockerfile:

```
# Use official Node.js image
FROM node:18

# Set the working directory
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose the port on which the app runs
EXPOSE 3000

# Command to run the application
CMD [ "npm", "start" ]
```

I created an image out of this Dockefile:

```
docker build -t fansari9993/test9:tagname .
```

![alt text](images/Day_7_Images/Image_5)

Then I pushed the image to my public repo using below command:

```
docker build -t fansari9993/test9:tagname
```

Now, below are my Kubernetes deployment files.
deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app-deployment
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
        image: your-dockerhub-username/nodejs-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: PORT
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: PORT
        - name: NODE_ENV
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: NODE_ENV
```
configmap.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  PORT: "3000"
```
secret.yaml:
```
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  NODE_ENV: cHJvZHVjdGlvbmFs # Base64 encoded value for "production"
```

Now, I deployed the files using below commands:
```
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml
```

![alt text](images/Day_7_Images/Image_6)

Below, is my Horizontal Pod Autoscaler file named hpa.yaml:

```
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: nodejs-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nodejs-app-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

```
kubectl apply -f kubernetes/hpa.yaml
```

As all the resources are successfully deployed we can check the status using below commands:
```
kubectl get pods
kubectl get svc
kubectl get hpa
```

![alt text](images/Day_7_Images/Image_8)

### To access the application:

```
kubectl expose deployment nodejs-app-deployment --type=NodePort --name=nodejs-app-service

minikube service nodejs-app-service --url
```
![alt text](images/Day_7_Images/Image_9)

In the below image, we can see the desired webpage deployed using above operation:

![alt text](images/Day_7_Images/Image_10)
