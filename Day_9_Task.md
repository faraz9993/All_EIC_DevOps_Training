# Day 9 Task
### - In this task I have deployed a front-end webpage, backend, ingress file, generated a self-signed certificate, sticky sessions and horizontal pod auto-scalling


First of all, I wrote the code for my webpage in HTML.
Below is my sample code:

```
<!DOCTYPE html>
<html>
<head>
    <title>This is a front end web page</title>
</head>
<body>
    <h1>This is a front-end web page</h1>
</body>
</html>
```
Then, I created the Dockerfile for this code:

```
FROM nginx:1.10.1-alpine
COPY index.html /usr/share/nginx/html
EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```
I created an image out of this Dockerfile and pushed it to my public repository:

```
docker build -t fansari9993/test9:v9 .
docker push fansari9993/test9:v9 
```
After that I created my deployment fies. They are as below.

frontend.yaml file:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-static-site
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-static-site
  template:
    metadata:
      labels:
        app: nginx-static-site
    spec:
      containers:
      - name: nginx-static-site
        image: fansari9993/test9:v9
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "500m"
          limits:
            cpu: "1"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-static-site-service
spec:
  selector:
    app: nginx-static-site
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
    nodePort: 30001
  type: NodePort
  ```

backend.yaml file:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args:
          - "-text=Hello from backend"
        ports:
        - containerPort: 5678
        resources:
          requests:
            cpu: "500m"
          limits:
            cpu: "1"
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 8008
    targetPort: 5678
    nodePort: 30002
  type: NodePort
```

ingress.yaml file:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-static-site-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "route"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - minikube.ip
    secretName: my-tls-secret
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /frontend
        pathType: Prefix
        backend:
          service:
            name: nginx-static-site-service
            port:
              number: 8080
      - path: /backend
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8008
```

Now, apply all these deployment files using below command:

```
kubectl apply -f frontend.yaml
kubectl apply -f backend.yaml
kubectl apply -f ingress.yaml
```

Next, I resolve the DNS of minikube ip with my webpage url. I made an enrry of it in /etc/hosts.

```
192.168.49.2    myapp.local
```

You can get the IP of your minikube using the below command:

```
minikube ip
```
After applying these three files, you must be able to get the webpage using the url as below.

![alt text](images/Day_9_Images/Image_1)

![alt text](images/Day_9_Images/Image_2)

Then I created the tls certificate and the tls key using below command:

```
kubectl create secret tls my-tls-secret --cert=tls.crt --key=tls.key
```

As soon as you will run this command two new files will be created in your account.

One with the name of tls.crt & another with the name of tls.key.

You will have to mention this certificate in your ingress file which I had already done. You can confirm it by running the command:

```
kubectl get secret
```
Next, step is to creating HorizontalPodApplication. Below is the file for that:

hpa.yaml:

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-static-site-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-static-site
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 6
```

You can apply this file using the command:

```
kubectl apply -f hpa.yaml
```
As soon as you will apply this file, You will be able to get the below output:

![alt text](images/Day_9_Images/Image_3)

One more thing needed here is the sticky session. We need to have the sticky session to keep a session active for a set amount of time.

In Kubernetes cluster, this means that all traffic from a particular client to an application is redirected to the same pod, even if the number of replicas is scaled up.

For that you need to add a few lines in the annotation:

```
nginx.ingress.kubernetes.io/affinity: "cookie"

nginx.ingress.kubernetes.io/session-cookie-name: "route"

nginx.ingress.kubernetes.io/rewrite-target: /

nginx.ingress.kubernetes.io/session-cookie-max-age: “1800”
```

This will keep the session active for 1800 seconds which is 30 mnts.


------

Now, the next step is testing.

You have to bring the traffic on your website so that if the CPU utilization goes beyond 6% the new pod is created or not.

For that I haved used the below script:

```
#!/bin/bash
# Variables
URL="https://myapp.local/frontend"
REQUESTS=3000
CONCURRENCY=10
echo "Sending traffic to $URL using ApacheBench"
siege -c$CONCURRENT_USERS -r$REQUESTS $URL
```

give the necessary execution permission to the file and run it.

As soon as, the traffic will start hitting the webpage, CPU utilization will increase and as soon as it will go beyond 6% the new pod will be created.

![alt text](images/Day_9_Images/Image_4)

![alt text](images/Day_9_Images/Image_5)

To fulfill this requirement, you must write resources in the deployment file.

```
ports:
- containerPort: 5678
resources:
  requests:
    cpu: "500m"
  limits:
    cpu: "1"
```
-------