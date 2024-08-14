# Day 25:
### In this task, I have deployed a web application on AWS using EC2 instances, configuring security groups and setting up an Application Load Balancer (ALB) with path-based routing.

### So, first of all, I created two ec2 instances Faraz_Instance_1 and Faraz_Instance_2 in AWS and configured webpages for them.

![alt text](images/Day_25_Images/Image_2)


### The webpages can be seen in the below images.
![alt text](images/Day_25_Images/Image_6)
![alt text](images/Day_25_Images/Image_7)

### Next, I created Application Load Balancer, named Faraz_LB and added two target groups, one for each instance.

![alt text](images/Day_25_Images/Image_8)

### I also made changes in the configuration file etc/apache2/sites-available/000-default.conf of the apache server of both the instances as below:

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

### Similar for the other instance.

### Furthermore, after the creation of the load balancer, I defined the rules as follow.

![alt text](images/Day_25_Images/Image_9)

### After, Successful configuration, the below url were redirected to their respective webpages.

![alt text](images/Day_25_Images/Image_11)
![alt text](images/Day_25_Images/Image_10)