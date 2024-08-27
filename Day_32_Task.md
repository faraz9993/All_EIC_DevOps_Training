# Day 32:
### In this task, I have configured an ECS cluster, defining task definitions, creating services and ensuring proper networking and security configurations using VPC, subnets, security groups and IAM roles using terraform.
### Below is my tree structure for all the files.:

```
.
├── main.tf
├── variables.tf
├── outputs.tf
├── ecs
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── iam
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── rds
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── sg
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── vpc
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf

```

### main.tf:
```
provider "aws" {
  region  = "us-east-2"
  version = "~> 5.0"
}

module "vpc" {
  source = "./vpc"

  cidr             = var.cidr
  public_subnets   = var.public_subnets
  private_subnets  = var.private_subnets
  availability_zones = var.availability_zones
}

module "security_groups" {
  source            = "./sg"
  vpc_id            = module.vpc.vpc_id
  container_port    = var.container_port
  name              = var.name
  environment       = var.environment
}

module "ecs" {
  source               = "./ecs"
  cluster_name         = var.cluster_name
  task_family_name     = var.task_family_name
  container_image      = var.container_image
  container_port       = var.container_port
  container_environment = var.container_environment
  ecs_service_name     = var.ecs_service_name
  alb_security_group_id = module.security_groups.alb_security_group_id
  vpc_id               = module.vpc.vpc_id
  public_subnets       = module.vpc.public_subnets
}

module "rds" {
  source                = "./rds"
  db_name               = var.db_name
  db_username           = var.db_username
  db_password           = var.db_password
  vpc_security_group_id = module.security_groups.rds_security_group_id
  private_subnets       = module.vpc.private_subnets
}

module "iam" {
  source                = "./iam"
}
```
### variables.tf:
```
variable "cidr" {
  description = "The CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnets" {
  description = "The list of public subnets CIDR blocks"
  type        = list(string)
  default     = ["10.0.3.0/24"]
}

variable "private_subnets" {
  description = "The list of private subnets CIDR blocks"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"] 
}

variable "availability_zones" {
  description = "The list of availability zones"
  type        = list(string)
  default     = ["us-east-2a", "us-east-2b"]
}

variable "name" {
  description = "The name prefix for resources"
  type        = string
  default     = "faraz"
}

variable "environment" {
  description = "The environment name (e.g., dev, prod)"
  type        = string
  default     = "dev"
}

variable "container_port" {
  description = "The port on which the container is running"
  type        = number
  default     = 80
}

variable "cluster_name" {
  description = "The name of the ECS cluster"
  type        = string
  default     = "Faraz_Cluster"
}

variable "task_family_name" {
  description = "The name of the ECS task family"
  type        = string
  default     = "Faraz_Cluster"
}

variable "container_image" {
  description = "The Docker image to be used"
  type        = string
  default     = "Faraz_Image"

}

variable "container_environment" {
  description = "The environment variables for the container"
  type = list(object({
    name  = string
    value = string
  }))
  default = [
    {
      name  = "ENVIRONMENT"
      value = "Dev"
    }
  ]
}

variable "ecs_service_name" {
  description = "The name of the ECS service"
  type        = string
  default     = "Faraz_ECS"
}

variable "db_name" {
  description = "The name of the database"
  type        = string
  default = "faraz-ecs-db"
}

variable "db_username" {
  description = "The username for the database"
  type        = string
  default = "admin"
}

variable "db_password" {
  description = "The password for the database"
  type        = string
  default = "MySecurePassword123!"
}


```

### outputs.tf:
```
output "vpc_id" {
  value = module.vpc.vpc_id
}

output "alb_security_group_id" {
  value = module.security_groups.alb_security_group_id
}

output "ecs_cluster_id" {
  value = module.ecs.ecs_cluster_id
}

output "rds_endpoint" {
  value = module.rds.rds_endpoint
}

```

### sg/main.tf:
```
resource "aws_security_group" "alb" {
  name   = "${var.name}-sg-alb-${var.environment}"
  vpc_id = var.vpc_id

  ingress {
    protocol         = "tcp"
    from_port        = 80
    to_port          = 80
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  ingress {
    protocol         = "tcp"
    from_port        = 443
    to_port          = 443
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  egress {
    protocol         = "-1"
    from_port        = 0
    to_port          = 0
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
}

resource "aws_security_group" "ecs_tasks" {
  name   = "${var.name}-sg-task-${var.environment}"
  vpc_id = var.vpc_id

  ingress {
    protocol         = "tcp"
    from_port        = var.container_port
    to_port          = var.container_port
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  egress {
    protocol         = "-1"
    from_port        = 0
    to_port          = 0
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
}

resource "aws_security_group" "rds" {
  name   = "${var.name}-sg-rds-${var.environment}"
  vpc_id = var.vpc_id

  ingress {
    protocol         = "tcp"
    from_port        = 3306
    to_port          = 3306
    cidr_blocks      = ["0.0.0.0/0"]
  }

  egress {
    protocol         = "-1"
    from_port        = 0
    to_port          = 0
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
}

```
### sg/variables.tf:
```
variable "vpc_id" {
  description = "The ID of the VPC"
  type        = string
}

variable "container_port" {
  description = "The port on which the container is running"
  type        = number
}

variable "name" {
  description = "The name prefix for resources"
  type        = string
}

variable "environment" {
  description = "The environment name (e.g., dev, prod)"
  type        = string
}

```

### sg/outputs.tf:
```
output "alb_security_group_id" {
  value = aws_security_group.alb.id
}

output "ecs_security_group_id" {
  value = aws_security_group.ecs_tasks.id
}

output "rds_security_group_id" {
  value = aws_security_group.rds.id
}
```

### iam/main.tf:
```
resource "aws_iam_role" "ecs_task_role" {
  name = "${var.name}-ecsTaskRole"
 
  assume_role_policy = <<EOF
{
 "Version": "2012-10-17",
 "Statement": [
   {
     "Action": "sts:AssumeRole",
     "Principal": {
       "Service": "ecs-tasks.amazonaws.com"
     },
     "Effect": "Allow",
     "Sid": ""
   }
 ]
}
EOF
}
 
resource "aws_iam_policy" "dynamodb" {
  name        = "${var.name}-task-policy-dynamodb"
  description = "Policy that allows access to DynamoDB"
 
 policy = <<EOF
{
   "Version": "2012-10-17",
   "Statement": [
       {
        "Effect": "Allow",
           "Action": [
               "dynamodb:CreateTable",
               "dynamodb:UpdateTimeToLive",
               "dynamodb:PutItem",
               "dynamodb:DescribeTable",
               "dynamodb:ListTables",
               "dynamodb:DeleteItem",
               "dynamodb:GetItem",
               "dynamodb:Scan",
               "dynamodb:Query",
               "dynamodb:UpdateItem",
               "dynamodb:UpdateTable"
           ],
           "Resource": "*"
       }
   ]
}
EOF
}
 
resource "aws_iam_role_policy_attachment" "ecs-task-role-policy-attachment" {
  role       = aws_iam_role.ecs_task_role.name
  policy_arn = aws_iam_policy.dynamodb.arn
}

resource "aws_iam_role" "ecs_task_execution_role" {
  name = "${var.name}-ecsTaskExecutionRole"
 
  assume_role_policy = <<EOF
{
 "Version": "2012-10-17",
 "Statement": [
   {
     "Action": "sts:AssumeRole",
     "Principal": {
       "Service": "ecs-tasks.amazonaws.com"
     },
     "Effect": "Allow",
     "Sid": ""
   }
 ]
}
EOF
}
 
resource "aws_iam_role_policy_attachment" "ecs-task-execution-role-policy-attachment" {
  role       = aws_iam_role.ecs_task_execution_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}
```
### iam/variables.tf:
```
variable "name" {
  description = "The name prefix for resources"
  type        = string
  default     = "faraz"
}
```

### vpc/main.tf:

```
resource "aws_vpc" "main" {
  cidr_block = var.cidr
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}

resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = element(var.private_subnets, count.index)
  availability_zone = element(var.availability_zones, count.index)
  count             = length(var.private_subnets)
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = element(var.public_subnets, count.index)
  availability_zone       = element(var.availability_zones, count.index)
  count                   = length(var.public_subnets)
  map_public_ip_on_launch = true
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route" "public" {
  route_table_id         = aws_route_table.public.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.main.id
}

resource "aws_route_table_association" "public" {
  count          = length(var.public_subnets)
  subnet_id      = element(aws_subnet.public.*.id, count.index)
  route_table_id = aws_route_table.public.id
}

resource "aws_nat_gateway" "main" {
  count         = length(var.private_subnets)
  allocation_id = element(aws_eip.nat.*.id, count.index)
  subnet_id     = element(aws_subnet.public.*.id, count.index)
  depends_on    = [aws_internet_gateway.main]
}

resource "aws_eip" "nat" {
  count = length(var.private_subnets)
  vpc   = true
}

resource "aws_route_table" "private" {
  count  = length(var.private_subnets)
  vpc_id = aws_vpc.main.id
}

resource "aws_route" "private" {
  count                  = length(compact(var.private_subnets))
  route_table_id         = element(aws_route_table.private.*.id, count.index)
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = element(aws_nat_gateway.main.*.id, count.index)
}

resource "aws_route_table_association" "private" {
  count          = length(var.private_subnets)
  subnet_id      = element(aws_subnet.private.*.id, count.index)
  route_table_id = element(aws_route_table.private.*.id, count.index)
}
```
### vpc/variables.tf:
```
variable "cidr" {
  description = "The CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnets" {
  description = "The list of public subnets CIDR blocks"
  type        = list(string)
  default     = ["10.0.3.0/24"]
}

variable "private_subnets" {
  description = "The list of private subnets CIDR blocks"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"] 
}

variable "availability_zones" {
  description = "The list of availability zones"
  type        = list(string)
  default     = ["us-east-2a", "us-east-2b"]
}

```

### vpc/outputs.tf:
```
output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnets" {
  value = aws_subnet.public[*].id
}

output "private_subnets" {
  value = aws_subnet.private[*].id
}
```

### ecs/main.tf:
```
resource "aws_ecs_cluster" "main" {
  name = var.cluster_name
}

resource "aws_ecs_task_definition" "main" {
  family                   = var.task_family_name
  container_definitions    = jsonencode([{
    name      = "my-container"
    image     = var.container_image
    essential = true
    portMappings = [{
      containerPort = var.container_port
      hostPort      = var.container_port
    }]
    environment = var.container_environment
  }])
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "256"
  memory                   = "512"
}

resource "aws_ecs_service" "main" {
  name            = var.ecs_service_name
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.main.arn
  desired_count   = 1
  launch_type     = "FARGATE"

  network_configuration {
    subnets         = var.public_subnets
    security_groups = [var.alb_security_group_id]
  }
}

resource "aws_alb_target_group" "main" {
  name     = "${var.name}-tg"
  port     = var.container_port
  protocol = "HTTP"
  vpc_id   = var.vpc_id
   target_type = "ip"

  health_check {
    path                = "/"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }
}

```
### ecs/variables.tf:
```
variable "name" {
  description = "The name of the ECS cluster"
  type        = string
  default     = "FarazService9"
}

variable "cluster_name" {
  description = "The name of the ECS cluster"
  type        = string
}

variable "task_family_name" {
  description = "The name of the ECS task family"
  type        = string
}

variable "container_image" {
  description = "The Docker image to be used"
  type        = string
}

variable "container_port" {
  description = "The port on which the container listens"
  type        = number
}

variable "container_environment" {
  description = "Environment variables for the container"
  type        = list(map(string))
}

variable "ecs_service_name" {
  description = "The name of the ECS service"
  type        = string
}

variable "alb_security_group_id" {
  description = "The security group ID for the ALB"
  type        = string
}

variable "vpc_id" {
  description = "The ID of the VPC"
  type        = string
}

variable "public_subnets" {
  description = "The public subnets to use"
  type        = list(string)
}
```

### ecs/outputs.tf:
```
output "ecs_cluster_id" {
  value = aws_ecs_cluster.main.id
}

output "ecs_service_id" {
  value = aws_ecs_service.main.id
}
```

### rds/main.tf:
```
resource "aws_db_subnet_group" "main" {
  name       = var.db_subnet_group_name
  subnet_ids = var.private_subnets

  tags = {
    Name = "RDS Subnet Group"
  }
}

resource "aws_db_instance" "main" {
  allocated_storage    = 20
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  username             = var.db_username
  password             = var.db_password
  db_subnet_group_name = aws_db_subnet_group.main.name
  vpc_security_group_ids = [var.vpc_security_group_id]
  skip_final_snapshot  = true

  tags = {
    Name = "MyRDSInstance"
  }
}
```
### rds/variables.tf:
```
variable "db_name" {
  description = "The name of the database"
  type        = string
  default     = "Faraz_DB"
}

variable "db_username" {
  description = "The username for the database"
  type        = string
}

variable "db_password" {
  description = "The password for the database"
  type        = string
}

variable "vpc_security_group_id" {
  description = "The VPC security group ID"
  type        = string
}

variable "private_subnets" {
  description = "The private subnets to use"
  type        = list(string)
}

variable "db_subnet_group_name" {
  type        = string
  default     = "faraz_subnet_group"
}
```
### rds/outputs.tf:
```
output "rds_endpoint" {
  value = aws_db_instance.main.endpoint
}
```

### After having these files as theire proper places as shown in the tree structure, in the above image, run below three commmands:
```
terraform init
terrafomr plan
terrafomr apply
``` 

### After runnning these three commands, the resources will be created as shown in below images:

![alt text](images/Day_32_Images/Image_1)
![alt text](images/Day_32_Images/Image_2)
![alt text](images/Day_32_Images/Image_3)
![alt text](images/Day_32_Images/Image_4)
![alt text](images/Day_32_Images/Image_5)
![alt text](images/Day_32_Images/Image_6)
![alt text](images/Day_32_Images/Image_7)
![alt text](images/Day_32_Images/Image_8)



