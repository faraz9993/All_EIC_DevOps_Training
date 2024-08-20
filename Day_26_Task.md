# Day 26:

### In this task, I have designed, deployed and managed a comprehensive and scalable Employee Web Application on AWS. The project integrates multiple AWS services, including S3, EC2, Auto Scaling, Load Balancer, VPC (without NAT Gateway) and RDS. The platform is highly available, secure and optimized for performance.

### So, first of all, I created a VPC named Faraz_VPC. The entire operation is done in us-west-2 region which is Oregon.

![alt text](images/Day_26_Images/Image_8)

### In that VPC, I created 4 subnets.
### 1. Faraz_Public_Subnet_1
### 2. Faraz_Public_Subnet_2
### 3. Faraz_Private_Subnet_1
### 4. Faraz_Priavte_Subnet_2

![alt text](images/Day_26_Images/Image_9)

### However, only naming subnets as public or private isn't gonna make it public or private. We have to configure subnet association with Internet Gateway using the route table, in order to make them public. So, I created two route tables as well.

### 1. Public Route Table
### 2. Private Route Table

### Next, I created an Internet Gateway first named "Faraz_IGW".

![alt text](images/Day_26_Images/Image_15)

### I did the subnet association of Faraz_Public_Subnet_1 and Faraz_Public_Subnet_2 with Internet Gateway in order to make them public as can be seen in the below image.

![alt text](images/Day_26_Images/Image_12)

### I also added this IGW route in the Puclic Route Table with destinition 0.0.0.0/0 . 

### The security group that I used is named as "Faraz_SG" and it has inbound rule of HTTP, HTTPS and SSH and outbound rule with "All traffic".

### Next, I created an S3 Bucket with the name "shopmax-static-content-faraz".

### The basic configuration of the S3 bucket is as below:

### 1. Bucket type: General Purpose
### 2. ACLs Enabled
### 3. Unselect "Block All Public Access"
### 4. Bucket Versioning: Enabled

### Once, S3 bucket was created, I created an EC2 instance where my Employee-Web-Application would run.

### Then, I created MySQL RDS. The basic configuration of the MySQL RDS is as below:
### 1. RDS: MySQL
### 2. DB Identifier: faraz-rds-db
### 3. DB Instance Size: db.t3.micro
### 4. VPC: Faraz_VPC
### 5. Subnet: Faraz_Private_Subnet_1

![alt text](images/Day_26_Images/Image_30)

### Next, I created an EC2 instance where my application will be running. The basic configuration of the EC2 instance is as below:

### 1. AMI: Amazon Linux
### 2. Instance type: t2.micro
### 3. VPC: Faraz_VPC
### 4. Subnet: Faraz_Public_Subnet_1
### 5. Auto-Assign Public IP: Enable
### 6. Below is my user-data that would connect the EC2 instance with the S3 bucket and MySQL RDS.
### User-data:

```
#!/bin/bash -ex
wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-GCNv2/FlaskApp.zip
unzip FlaskApp.zip
cd FlaskApp/
yum -y install python3-pip
pip install -r requirements.txt
yum -y install stress
export PHOTOS_BUCKET=shopmax-static-content-faraz
export AWS_DEFAULT_REGION= us-west-2
export MYSQL_HOST=localhost
export MYSQL_USER=admin
export MYSQL_PASSWORD=XXXXXXXX
export MYSQL_DB=faraz-rds-db
FLASK_APP=application.py /usr/local/bin/flask run --host=0.0.0.0 --port=80
```

![alt text](images/Day_26_Images/Image_27)

### Furtermore, I also created a Load Balancer and the Auto Scalling Group for the purpose of high availablity of the application.

### For load balancer, I selected an Application Load Balancer and below is my Target GRoup for that.

![alt text](images/Day_26_Images/Image_31)

### For Auto Scalling Group, I created a launch template. The basic configuration of the Launch Template is as below.

### 1. AMI ID: ami-0a38c1c38a15a15fed74
### 2. Instance type: t2.micro
### 3. Key pair name: Faraz_Tak_Key
### 4. Security Group: Faraz_SG


### The basic configuration of the Auto Scalling Group is as below.

### 1. Auto Scalling Group Name: Faarz-ASG
### 2. Desired Capacity: 3
### 3. Minimum Capacity: 1
### 4. Maximum Capacity: 5

### After the successful deployment of all the above mentioned resources, I was able to get my Employee-web-application as can be seen in the below image after hitting the public IP of the instance, I had created earlier:

![alt text](images/Day_26_Images/Image_35)

### When, we would click on the "Add" button, we will be taken to the below page:

![alt text](images/Day_26_Images/Image_34)

### After, filling-up the mentioned detils, we can see that the uploaded image is saved in the S3 Bucket named "shopmax-static-conetnt-faraz".

![alt text](images/Day_26_Images/Image_36)

### The structured data such as Full name, Location, the Job title and badges will be saved in the MySQL Database. 









