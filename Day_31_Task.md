# Day 31:

### In this task, I have deployed a complete infrastructure using Terraform, emphasizing the usage of state lock, variables, .tfvars files, modules, functions, workspaces, and lifecycle rules.

### Below are my terraform files:

### So, first of all, there is main.tf file of the root module directory where all the modules are:

### main.tf:

```
module "vpc" {
  source             = "./vpc"
  vpc_cidr           = "10.0.0.0/16"
  public_subnets     = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets    = ["10.0.3.0/24", "10.0.4.0/24"]
  availability_zones = ["us-east-2a", "us-east-2b"]
}

module "ec2_instance" {
  source              = "./ec2"
  ami                 = "ami-0c55b159cbfafe1f0"
  instance_type       = var.instance_type
  key_name            = "Faraz_Key"
  subnet_id           = module.vpc.public_subnet_ids[0]
  security_group_ids  = [module.security_group.security_group_id]
  instance_name       = var.instance_name
  additional_tags     = {
    Environment = "default"
  }
  instance_profile_name = "my_instance_profile_9"
  role_name             = module.IAM.role_arn
}

module "s3_bucket" {
  source          = "./s3"
  bucket_name     = var.bucket_name
  bucket_acl      = var.bucket_acl
  versioning_enabled = var.versioning_enabled
  sse_algorithm   = var.sse_algorithm
  additional_tags = {
    Environment = "staging"
  }
}

module "security_group" {
  source              = "./sg"
  name                = "web_sg"
  description         = "Security group for web instances"
  vpc_id              = module.vpc.vpc_id
  ingress_from_port   = 22
  ingress_to_port     = 22
  ingress_protocol    = "tcp"
  ingress_cidr_blocks = ["0.0.0.0/0"]
  egress_from_port    = 0
  egress_to_port      = 0
  egress_protocol     = "tcp"
  egress_cidr_blocks  = ["0.0.0.0/0"]
  additional_tags     = {
    Environment = "staging"
  }
}

module "rds" {
  source                = "./rds"
  allocated_storage     = 20
  engine                = "mysql"
  engine_version        = "8.0"
  instance_class        = "db.t3.micro"
  db_name               = "mydatabase"
  username              = "admin"
  password              = "mysecretpassword"
  db_subnet_group_name  = module.vpc.db_subnet_group_name
  vpc_security_group_ids = [module.security_group.security_group_id]
  tags                  = {
    Name = "My RDS instance"
  }
}

module "dynamodb" {
  source    = "./dynamo"
}

module "IAM" {
  source      = "./IAM"
  role_name   = "s3rds_accessrole"
  bucket_name = "kauserbucket"
  tags = {
    Environment = "staging"
  }
}
```
### variables.tf:

```
variable "bucket_name" {
  description = "The name of the S3 bucket"
  type        = string
  default     = "kauserbucket"
}

variable "bucket_acl" {
  description = "The ACL for the S3 bucket"
  type        = string
  default     = "private"
}

variable "versioning_enabled" {
  description = "Enable versioning for the S3 bucket"
  type        = bool
  default     = true
}

variable "sse_algorithm" {
  description = "The server-side encryption algorithm to use"
  type        = string
  default     = "AES256"
}

variable "instance_type" {
  description = "Type of the EC2 instance (e.g., t2.micro)"
  type        = string
  default     = "t2.micro"
}

variable "instance_name" {
  description = "Type of the EC2 instance (e.g., t2.micro)"
  type        = string
  default     = "default_ws_instance"
}
```

### outputs.tf:
```
output "s3_bucket_name" {
  description = "Name of the S3 bucket"
  value       = module.s3_bucket.bucket_name
}

output "s3_bucket_arn" {
  description = "ARN of the S3 bucket"
  value       = module.s3_bucket.bucket_arn
}
```

### vpc/main.tf:
```
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name = "MainVPC"
  }
}

resource "aws_subnet" "public" {
  count = length(var.public_subnets)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnets[count.index]
  map_public_ip_on_launch = true
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "PublicSubnet-${count.index}"
  }
}

resource "aws_subnet" "private" {
  count = length(var.private_subnets)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnets[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "PrivateSubnet-${count.index}"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "InternetGateway"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "PublicRouteTable"
  }
}

resource "aws_route_table_association" "public_association" {
  count          = length(aws_subnet.public)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_db_subnet_group" "db_subnet_group" {
  name       = "my-db-subnet-group"
  subnet_ids = aws_subnet.private[*].id

  tags = {
    Name = "My DB Subnet Group"
  }
}
```

### vpc/variables.tf:
```
variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnets" {
  description = "List of public subnet CIDR blocks"
  type        = list(string)
}

variable "private_subnets" {
  description = "List of private subnet CIDR blocks"
  type        = list(string)
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

```

### vpc/outputs.tf:
```
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "IDs of the private subnets"
  value       = aws_subnet.private[*].id
}

output "db_subnet_group_name" {
  value = aws_db_subnet_group.db_subnet_group.name
}

```

### sg/main.tf:
```
resource "aws_security_group" "this" {
  name        = var.name
  description = var.description
  vpc_id      = var.vpc_id

  ingress {
    from_port   = var.ingress_from_port
    to_port     = var.ingress_to_port
    protocol    = var.ingress_protocol
    cidr_blocks = var.ingress_cidr_blocks
  }

  egress {
    from_port   = var.egress_from_port
    to_port     = var.egress_to_port
    protocol    = var.egress_protocol
    cidr_blocks = var.egress_cidr_blocks
  }

  tags = merge({
    Name = var.name
  }, var.additional_tags)
}

```

### sg/variables.tf:
```
variable "name" {
  description = "Name of the security group"
  type        = string
}

variable "description" {
  description = "Description of the security group"
  type        = string
}

variable "vpc_id" {
  description = "VPC ID where the security group will be created"
  type        = string
}

variable "ingress_from_port" {
  description = "The start port for inbound traffic"
  type        = number
  default     = 0
}

variable "ingress_to_port" {
  description = "The end port for inbound traffic"
  type        = number
  default     = 0
}

variable "ingress_protocol" {
  description = "The protocol for inbound traffic"
  type        = string
  default     = "tcp"
}

variable "ingress_cidr_blocks" {
  description = "The CIDR blocks for inbound traffic"
  type        = list(string)
  default     = ["0.0.0.0/0"]
}

variable "egress_from_port" {
  description = "The start port for outbound traffic"
  type        = number
  default     = 0
}

variable "egress_to_port" {
  description = "The end port for outbound traffic"
  type        = number
  default     = 0
}

variable "egress_protocol" {
  description = "The protocol for outbound traffic"
  type        = string
  default     = "tcp"
}

variable "egress_cidr_blocks" {
  description = "The CIDR blocks for outbound traffic"
  type        = list(string)
  default     = ["0.0.0.0/0"]
}

variable "additional_tags" {
  description = "Additional tags to apply to the security group"
  type        = map(string)
  default     = {}
}

```
### sg/outputs.tf:
```
output "security_group_id" {
  description = "ID of the security group"
  value       = aws_security_group.this.id
}

output "security_group_name" {
  description = "Name of the security group"
  value       = aws_security_group.this.name
}

```

### IAM/main.tf:
```
resource "aws_iam_role" "this" {
  name = var.role_name

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Allow",
        Principal = {
          Service = "ec2.amazonaws.com"
        },
        Action = "sts:AssumeRole"
      }
    ]
  })

  tags = var.tags
}

resource "aws_iam_policy" "s3_rds_policy" {
  name        = "S3RDSAccessingPolicy"
  description = "Policy to allow access to S3 and RDS"

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Allow",
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:ListBucket"
        ],
        Resource = [
          "arn:aws:s3:::${var.bucket_name}",
          "arn:aws:s3:::${var.bucket_name}/*"
        ]
      },
      {
        Effect = "Allow",
        Action = [
          "rds:DescribeDBInstances",
          "rds:Connect"
        ],
        Resource = "*"
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "this" {
  role       = aws_iam_role.this.name
  policy_arn  = aws_iam_policy.s3_rds_policy.arn
}

output "role_arn" {
  value = aws_iam_role.this.arn
}

```
### IAM/variables.tf:
```
variable "role_name" {
  description = "The name of the IAM role."
  type        = string
}

variable "bucket_name" {
  description = "The name of the S3 bucket."
  type        = string
}

variable "tags" {
  description = "Tags to apply to the IAM role."
  type        = map(string)
  default     = {}
}

```
### ec2/main.tf:
```
resource "aws_instance" "this" {
  ami                    = var.ami
  instance_type          = var.instance_type
  key_name               = var.key_name
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids

  tags = merge({
    Name = var.instance_name
  }, var.additional_tags)

  lifecycle {
    create_before_destroy = true
  }

}

resource "aws_eip" "this" {
  instance = aws_instance.this.id
}

output "public_ip" {
  value = aws_eip.this.public_ip
}
```
### for EC2 instance, I have used life-cycle rule of create before destroy which will create a fresh  ec2 instance before destroying it.

### ec2/variables.tf:
```
variable "ami" {
  description = "AMI to be used for the instance"
  type        = string
  default     = "ami-0c55b159cbfafe1f0"
}

variable "instance_type" {
  description = "Type of the instance (e.g., t2.micro)"
  type        = string
  default     = "t2.micro"
}

variable "key_name" {
  description = "Name of the SSH key to use"
  type        = string
}

variable "subnet_id" {
  description = "Subnet ID where the instance will be placed"
  type        = string
}
resource "aws_security_group" "this" {
  name        = var.name
  description = var.description
  vpc_id      = var.vpc_id

  ingress {
    from_port   = var.ingress_from_port
    to_port     = var.ingress_to_port
    protocol    = var.ingress_protocol
    cidr_blocks = var.ingress_cidr_blocks
  }

  egress {
    from_port   = var.egress_from_port
    to_port     = var.egress_to_port
    protocol    = var.egress_protocol
    cidr_blocks = var.egress_cidr_blocks
  }

  tags = merge({
    Name = var.name
  }, var.additional_tags)
}

variable "security_group_ids" {
  description = "List of security group IDs to assign to the instance"
  type        = list(string)
}

variable "instance_name" {
  description = "Name tag for the instance"
  type        = string
  default     = "default_ws_EC2Instance"
}

variable "additional_tags" {
  description = "Additional tags to apply to the instance"
  type        = map(string)
  default     = {}
}

variable "private_key_path" {
  description = "Additional tags to apply to the instance"
  type        = string
  default     = "/home/einfochips/Downloads/Faraz_Key_1.pem"
}


variable "instance_profile_name" {
  description = "The name of the IAM instance profile."
  type        = string
}

variable "role_name" {
  description = "The name of the IAM role to attach."
  type        = string
}
```

### ec2/outputs.tf:
```
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.this.id
}

output "public_ip2" {
  description = "Public IP address of the EC2 instance"
  value       = aws_eip.this.public_ip
}

```

### s3/main.tf:
```
resource "aws_s3_bucket" "bucket" {
  bucket = var.bucket_name
  acl    = var.bucket_acl

  tags = merge({
    Name = var.bucket_name
  }, var.additional_tags)
}

resource "aws_s3_bucket_versioning" "versioning" {
  bucket = aws_s3_bucket.bucket.id 

  versioning_configuration {
    status = var.versioning_enabled ? "Enabled" : "Suspended"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "sse" {
  bucket = aws_s3_bucket.bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = var.sse_algorithm
    }
  }
}

resource "aws_s3_bucket_public_access_block" "public_access_block" { 
  bucket = aws_s3_bucket.bucket.id  

  block_public_acls   = true
  block_public_policy = true
  ignore_public_acls  = true
  restrict_public_buckets = true
}

```
### s3/variables.tf:
```
variable "bucket_name" {
  description = "The name of the S3 bucket"
  type        = string
  default = "dev_kauserbucket"
}

variable "bucket_acl" {
  description = "The ACL for the S3 bucket"
  type        = string
  default     = "private"
}

variable "versioning_enabled" {
  description = "Enable versioning for the S3 bucket"
  type        = bool
  default     = true
}

variable "sse_algorithm" {
  description = "The server-side encryption algorithm to use"
  type        = string
  default     = "AES256"
}

variable "additional_tags" {
  description = "Additional tags to apply to the S3 bucket"
  type        = map(string)
  default     = {}
}

```

### s3/outputs.tf:
```
output "bucket_name" {
  description = "The name of the S3 bucket"
  value       = aws_s3_bucket.bucket.bucket
}

output "bucket_arn" {
  description = "The ARN of the S3 bucket"
  value       = aws_s3_bucket.bucket.arn
}

```

### dynamo/main.tf:
```
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "faraz-dynamo-db"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name = "Terraform Lock Table"
  }
}

```

### rds/main.tf:
```
resource "aws_db_instance" "this" {
  allocated_storage     = var.allocated_storage
  engine                = var.engine
  engine_version        = var.engine_version
  instance_class        = var.instance_class
  username              = var.username
  password              = var.password
  parameter_group_name  = var.parameter_group_name
  skip_final_snapshot   = var.skip_final_snapshot
  publicly_accessible   = var.publicly_accessible
  db_subnet_group_name  = var.db_subnet_group_name
  vpc_security_group_ids = var.vpc_security_group_ids

  tags = var.tags
}

lifecycle {
    prevent_destroy = true
  }
```

### I have used life-cycle rule of prevent_destroy for rds.

### rds/variables.tf:
```
variable "allocated_storage" {
  description = "The allocated storage in gigabytes."
  type        = number
  default     = 20
}

variable "engine" {
  description = "The database engine to use."
  type        = string
  default     = "mysql"
}

variable "engine_version" {
  description = "The version of the database engine."
  type        = string
  default     = "8.0"
}

variable "instance_class" {
  description = "The instance type of the RDS instance."
  type        = string
  default     = "db.t3.micro"
}

variable "db_name" {
  description = "The name of the database."
  type        = string
  default     = "mydatabase"
}

variable "username" {
  description = "Username for the master DB user."
  type        = string
  default     = "admin"
}

variable "password" {
  description = "Password for the master DB user."
  type        = string
  sensitive   = true
}

variable "parameter_group_name" {
  description = "Name of the DB parameter group to associate."
  type        = string
  default     = "default.mysql8.0"
}

variable "skip_final_snapshot" {
  description = "Determines whether a final DB snapshot is created before the DB instance is deleted."
  type        = bool
  default     = true
}

variable "publicly_accessible" {
  description = "Whether the DB should be publicly accessible."
  type        = bool
  default     = false
}

variable "db_subnet_group_name" {
  description = "Name of DB subnet group to use for the instance."
  type        = string
  default     = "module.vpc.db_subnet_group_name"
}

variable "vpc_security_group_ids" {
  description = "List of VPC security groups to associate with the RDS instance."
  type        = list(string)
  default     = []
}

variable "tags" {
  description = "A map of tags to assign to the resource."
  type        = map(string)
  default     = {}
}

```

### rds/outputs.tf:
```
output "db_instance_id" {
  description = "The RDS instance ID."
  value       = aws_db_instance.this.id
}

output "db_instance_endpoint" {
  description = "The connection endpoint."
  value       = aws_db_instance.this.endpoint
}

output "db_instance_arn" {
  description = "The ARN of the RDS instance."
  value       = aws_db_instance.this.arn
}

```

### After having these many files at their proper places as it appears in the tree, we have to run terraform commands at the root module dircetory:

```
terraform init
terraform plan
terraform apply
```
### As soon as, I run these three commands the resources were created on the AWS cloud platform as can be seen in the below image:

### ec2 instance:
![alt text](images/Day_31_Images/Image_1)

### s3 bucket:
![alt text](images/Day_31_Images/Image_2)

### IAM role for ec2 instace to use s3 bucket and rds:
![alt text](images/Day_31_Images/Image_3)

### RDS database:
![alt text](images/Day_31_Images/Image_4)

### DynamoDB Table:
![alt text](images/Day_31_Images/Image_5)

### As S3 bucket and DynamoDB tables are created, I created backend.tf file to save the terraform.tfstate file in S3 bucket lockid in to the DynamoDB tables:

### backend.tf file:
```
terraform {
  backend "s3" {
    bucket = "kauserbucket"
    key    = "terraform.tfstate"
    region = "us-east-2"
    dynamodb_table = "faraz-dynamo-db"
    encrypt        = true
  }
}
```

### terraform.tfstate file is saved in s3 bucket as can be seen in the below image: 
![alt text](images/Day_31_Images/Image_6)

### lockid is logged in the Dynamodb as can be seen in the below image:
![alt text](images/Day_31_Images/Image_7)

### Next, I created two IAM Users on AWS:
### 1. dev_user
### 2. prod_user

### I created two workspaces as well.
### 1. staging
### 2. production

### First, I worked on workspace named "staging" with user named "dev_user":
![alt text](images/Day_31_Images/Image_8)

### I configured aws using the access key and secret access key of the dev_user. For that I used below command:

```
aws configure --profile staging
```
![alt text](images/Day_31_Images/Image_9)

### Even for the dev_user the entire configurations is same, I just created a variables file named dev.tfvars and entered the necessary variables in it. This file is in root dircetory:

### dev.tfvars:

```
bucket_name         = "day31-staging-kauserbucket"
bucket_acl          = "private"
versioning_enabled  = true
sse_algorithm       = "AES256"
instance_type       = "t2.micro"
instance_name       = "day31-staging_ws_instance"
```

### After creating this file I ran the terraform apply command with below flag:
```
terraform apply -var-file="dev.tfvars"
```

### The needed resources were created om AWS cloud platform as can be seen in the below image:

![alt text](images/Day_31_Images/Image_11)

![alt text](images/Day_31_Images/Image_12)

![alt text](images/Day_31_Images/Image_13)


### Then, I started the work on prod environment. Similarly, as default configuartion and the staging configuration, I created the workspace for the productoin environment.

### I created a user named prod_user, created its access key and secret access key and logged in using its credentials using below command:

```
aws configure --profile production
```

![alt text](images/Day_31_Images/Image_14)

### Furthermore, I also created a variables file named prod.tfvars:

### prod.tfvars:
```
bucket_name         = "day31-prod-kauserbucket"
bucket_acl          = "private"
versioning_enabled  = true
sse_algorithm       = "AES256"
instance_type       = "t2.micro"
instance_name       = "day31-prod_ws_instance"
```
### I ran the below command, to create the resources within the prodiction environment:
```
terraform apply -var-file="prod.tfvars"
```

![alt text](images/Day_31_Images/Image_16)

![alt text](images/Day_31_Images/Image_17)

![alt text](images/Day_31_Images/Image_18)
