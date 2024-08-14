# Day 25:
### In this task, I have deployed a web application on AWS using EC2 instances, configuring security groups and setting up an Application Load Balancer (ALB) with path-based routing.

### So, first of all, I created two ec2 instances Faraz_Instance_1 and Faraz_Instance_2 in AWS and configured webpages for them.

![alt text](images/Day_25_Images/Image_1)


### The webpages can be seen in the below images using the public IP of both the instances.
![alt text](images/Day_25_Images/Image_6)
![alt text](images/Day_25_Images/Image_7)

### Next, I created Application Load Balancer, named Faraz_LB and added two target groups, one for each instance.

![alt text](images/Day_25_Images/Image_8)

### I created below directories for the purpose of meeting the task requirements:

### For instance 1:
```
mkdir -p /var/www/html/app1/index.html
```

### For instance 2:
```
mkdir -p /var/www/html/app2/index.html
```

### After the change, I restarted the service using below command.

```
systemctl restart apache2.service
```

### Furthermore, after the creation of the load balancer, I defined the rules as follow.

![alt text](images/Day_25_Images/Image_9)

### After the successful configuration, the below urls were showing their respective webpages.

![alt text](images/Day_25_Images/Image_11)
![alt text](images/Day_25_Images/Image_10)