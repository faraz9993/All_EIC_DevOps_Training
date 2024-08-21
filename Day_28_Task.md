### In this task, I have deployed a multi-tier architecture application on AWS using Terraform. The deployment involves Terraform variables, outputs and change sets. The multi-tier architecture includes an EC2 instance, an RDS MySQL DB instance and an S3 bucket.

### So, first of all, I configured AWS with my local machine so that resources can be created on the cloud platform.

```
aws configure


AWS Access Key ID:
AWS Secret Access Key:
Default region name: us-east-2
Default output format: json
```
### After that I created terraform main.tf file which is as per below:

```
provider "aws" {
  region = var.aws_region
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table_association" "public_rt_assoc" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-2a"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "public_b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-2b"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "private_a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.3.0/24"
  availability_zone = "us-east-2a"
}

resource "aws_subnet" "private_b" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.5.0/24"
  availability_zone = "us-east-2b"
}

resource "aws_security_group" "ec2_sg" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "rds_sg" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_db_subnet_group" "default" {
  name       = "farazdbsg"
  subnet_ids = [aws_subnet.private_a.id, aws_subnet.private_b.id]

  tags = {
    Name = "RDS subnet group"
  }
}

resource "aws_instance" "web" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public_a.id
  vpc_security_group_ids = [aws_security_group.ec2_sg.id]
  associate_public_ip_address = true

  tags = {
    Name = "Faraz-Server"
  }
}

resource "aws_db_instance" "mysql" {
  identifier              = "terraform-mysql"
  engine                  = "mysql"
  instance_class          = "db.t3.micro"
  allocated_storage       = 20
  username                = var.db_username
  password                = var.db_password
  vpc_security_group_ids  = [aws_security_group.rds_sg.id]
  db_subnet_group_name    = aws_db_subnet_group.default.name
  publicly_accessible     = false
  skip_final_snapshot     = true

  tags = {
    Name = "MySQLDB"
  }
}

resource "aws_s3_bucket" "bucket" {
  bucket = "kauserbucket"
  acl    = "private"
}

resource "aws_iam_role" "ec2_role" {
  name = "ec2-role"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
}

resource "aws_iam_role_policy" "ec2_policy" {
  name   = "ec2-policy"
  role   = aws_iam_role.ec2_role.id

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "${aws_s3_bucket.bucket.arn}/*"
      ]
    }
  ]
}
EOF
}

resource "aws_iam_instance_profile" "ec2_instance_profile" {
  name = "ec2-instance-profile"
  role = aws_iam_role.ec2_role.name
}
```
### Below is my variables.tf file:

```
variable "aws_region" {
  description = "The AWS region to deploy resources in"
  type        = string
  default     = "us-east-2"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
  default     = "ami-0862be96e41dcbf74"
}

variable "db_name" {
  description = "The name of the database"
  type        = string
  default     = "Faraz-DB"
}

variable "db_username" {
  description = "The username for the database"
  type        = string
  default     = "XXXXXXXXX" # Enter your username for the database   here.
}

variable "db_password" {
  description = "The password for the database"
  type        = string
  sensitive   = true
  default     = "XXXXXXXXX" # Enter your Password here.
}
```
### As per above terraform main.tf and variables.tf file a VPC will be created in which there would be 4 subnets, 2 public and 2 private. Out of two public subnets, one will have an EC2 instance and out of two private subnets one will have an RDS.

### As soon as, we will run terraform init, terraform plan and terraform apply command a terraform.tfstate file will be created by default and it is basically a JSON file that stores information about the infrastructure and the configuration.

### The resources will get created with the configurations mentioned as above.

### Below is the image of WebServer get created from main.tf file:

![alt text](images/Day_28_Images/Image_1)

### For the change set, I made a minor change in the Terraform configuration, such as modifying an EC2 instance tag from WebServer to Faraz-Server and it did reflected on the cloud as can be seen in the below image.

![alt text](images/Day_28_Images/Image_2)

### I have kept MySQL RDS in the private subnet so no other instance out of the VPC will be able to connect with the RDS only the instances within that VPC will get connect with the RDS. So, for the testing purpose, I tried connecting to the instance using RDS which did get connect as can be seen in the below image:

![alt text](images/Day_28_Images/Image_4)

### The command I entered to get connected is below:

```
mysql -h <Database_Endpoint> -u <User_Name> -p
```

### Next, I tested that whether EC2 instance can read and write to the S3 bucket or not. Turned out, EC2 instance do have the read/write permission for the S3 bucket as can be seen in the below image.

### The commands I ran in the EC2 instance for the testing-purpose are as below:

```
echo "Hello There" > testfile.txt
aws s3 cp testfile.txt s3://kauserbucket/
aws s3 ls s3://kauserbucket/
aws s3 rm s3://kauserbucket/testfile.txt
```
![alt text](images/Day_28_Images/Image_6)