# Day 35:

### In this task, I have deployed a Kubernetes application on AWS EC2 instances and set up a monitoring stack using Prometheus and Grafana. This project is designed to test the knowledge of deploying and configuring monitoring solutions in a Kubernetes environment on AWS

### For this task, First of all, I created 3 EC2 instances. One for control plane which was t2.medium and a couple for data plane which were t2.micro.

![alt text](images/Day_35_Images/Image_1)

### I ran a set of commands two make t2.medium control plane and t2.micro data planes.

![alt text](images/Day_35_Images/Image_2)
![alt text](images/Day_35_Images/Image_3)
![alt text](images/Day_35_Images/Image_4)

### When all the nodes were ready and joined using kubeadm cluster. I intalled prometheus and grafana using the helm chart.

### For installing prometheus using helm:

```
git clone https://github.com/nkheria/charts

vi prometheus-values.yml

alertmanager:
  persistentVolume:
    enabled: false
server:
  persistentVolume:
    enabled: false

kubectl create namespace prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus -f ~/prometheus-values.yml --namespace prometheus
kubectl get all -n prometheus
```

### After running the above commands, Prometheus was succesfully deployed on the kubeadm cluster as can be seen in the below images:

![alt text](images/Day_35_Images/Image_6)
![alt text](images/Day_35_Images/Image_7)

### Next, I installed grafana as using helm. For that, I ran the below given commands:

```
vi grafana-values.yml

adminPassword: password


kubectl create namespace grafana
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana -f ~/grafana-values.yml --namespace grafana
kubectl get all -n grafana

```
### To deploy a NodePort Service to Provide External Access to Grafana.
```
vi grafana-ext.yml

kind: Service
apiVersion: v1
metadata:
  namespace: grafana
  name: grafana-ext
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
  - protocol: TCP
    port: 3000
    nodePort: 30009

kubectl apply -f ~/grafana-ext.yml

```


### After running the above commands, the Grafana was succesfully deployed as can be seen in the below image. 

![alt text](images/Day_35_Images/Image_9)
![alt text](images/Day_35_Images/Image_12)

![alt text](images/Day_35_Images/Image_13)





