Below is my stack.yaml file which will create resources such as VPC, Subnet, Route table, EC2 Instance, S3 Bucket and the RDS.

```
Parameters:
  VpcCidr:
    Type: String
    Default: 10.0.0.0/16
    Description: CIDR block for the VPC.

  PublicSubnetCidr:
    Type: String
    Default: 10.0.1.0/24
    Description: CIDR block for the public subnet.

  PrivateSubnetCidr:
    Type: String
    Default: 10.0.2.0/24
    Description: CIDR block for the private subnet.


  InstanceType:
    Type: String
    Default: t2.micro
    Description: EC2 instance type.

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair.

  DBUsername:
    Type: String
    Default: admin
    NoEcho: true
    Description: The database admin account username.

  DBPassword:
    Type: String
    NoEcho: true
    Description: The database admin account password.
    MinLength: 6
    MaxLength: 41
    AllowedPattern: '[a-zA-Z0-9]*'
    ConstraintDescription: Must contain only alphanumeric characters.

  DBAllocatedStorage:
    Type: Number
    Default: 20
    Description: The size of the database (Gb).

  AMIId:
    Type: String
    Description: The AMI ID for the EC2 instance.
    Default: 'ami-0862be96e41dcbf74' # Replace with a region-specific AMI

  S3BucketName:
    Type: String
    Description: Unique name for the S3 bucket.

Resources:
  # VPC
  CustomVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCidr
      EnableDnsSupport: 'true'
      EnableDnsHostnames: 'true'
      Tags:
      - Key: Name
        Value: CustomVPC

  # Internet Gateway
  MyInternetGateway:
    Type: AWS::EC2::InternetGateway

  # Attach Internet Gateway to VPC
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref CustomVPC
      InternetGatewayId: !Ref MyInternetGateway

  # Public Subnet
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CustomVPC
      CidrBlock: !Ref PublicSubnetCidr
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: 'true'
      Tags:
      - Key: Name
        Value: PublicSubnet

  # Private Subnet
  PrivateSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CustomVPC
      CidrBlock: !Ref PrivateSubnetCidr
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
      - Key: Name
        Value: PrivateSubnet

    # Private Subnet 2
  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CustomVPC
      CidrBlock: 10.0.3.0/24 # Adjust the CIDR block for the second subnet
      AvailabilityZone: !Select [1, !GetAZs '']
      Tags:
      - Key: Name
        Value: PrivateSubnet2

  # Route Table for Public Subnet
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref CustomVPC
      Tags:
      - Key: Name
        Value: PublicRouteTable

  # Route to Internet Gateway for Public Subnet
  PublicRoute:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref MyInternetGateway

  # Associate Public Subnet with Route Table
  PublicSubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

  # Security Group for EC2 instance
  EC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP access on the inbound.
      VpcId: !Ref CustomVPC
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '80'
        ToPort: '80'
        CidrIp: 0.0.0.0/0

  # Security Group for RDS instance
  RDSSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow MySQL access only from the EC2 instance.
      VpcId: !Ref CustomVPC
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '3306'
        ToPort: '3306'
        SourceSecurityGroupId: !Ref EC2SecurityGroup

  # EC2 Instance
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      SecurityGroupIds:
      - !Ref EC2SecurityGroup
      SubnetId: !Ref PublicSubnet
      # IamInstanceProfile: !Ref EC2InstanceProfile
      ImageId: !Ref AMIId
      UserData: !Base64
        Fn::Sub: |
          #!/bin/bash
          apt update -y
          apt install -y apache2 mysql-client
          systemctl start apache2
          systemctl enable apache2

  # S3 Bucket
  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref S3BucketName

  # RDS MySQL DB Instance
  MyRDSInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: db.t3.micro
      Engine: MySQL
      MasterUsername: !Ref DBUsername
      MasterUserPassword: !Ref DBPassword
      AllocatedStorage: !Ref DBAllocatedStorage
      DBSubnetGroupName: !Ref DBSubnetGroup
      VPCSecurityGroups:
      - !Ref RDSSecurityGroup

  # DB Subnet Group
  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: Subnet group for RDS
      SubnetIds:
      - !Ref PrivateSubnet
      - !Ref PrivateSubnet2

```

Below are the images of teh resources created using above stack.yaml file:

![alt text](images/Day_27_Images/Image_1)

![alt text](images/Day_27_Images/Image_2)

![alt text](images/Day_27_Images/Image_3)

![alt text](images/Day_27_Images/Image_4)

![alt text](images/Day_27_Images/Image_5)

![alt text](images/Day_27_Images/Image_6)

![alt text](images/Day_27_Images/Image_7)