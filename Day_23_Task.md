# Day 23:

### In this task, I have created an IAM policy, attched that policy to an IAM role and also attched that policy to the IAM user. Later, I also estimated cost of 3 AWS services per annum.

### The policy I created was in JSON format. The name I gave to the policy is DevTeamPolicy. Below is that JSON formatted policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::application-logs"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::application-logs/*"
        }
    ]
}
```

### I attached that policy with the IAM role named 'DevTeamRole' as can be seen in the below image:

![alt text](images/Day_23_Images/Image_2)

### Next, I created a user named Test_User and attched the created policy with that user.

![alt text](images/Day_23_Images/Image_4)

### As per the policy, when the IAM user tried to list all the S3 buckets the permission was denied as can be seen in the below image.

![alt text](images/Day_23_Images/Image_5)

### Furthermore, I also created an estimation on AWS pricing calculator service named "My Estimate" which gave the estimation of three services.
### 1. Elastic Load Balancer 2. Amazon EC2 instance of t3.medium 3. Amazon RDS of db.m5.large. The estimation was for a year. The total estimated cost was 18,208 USD as can be seen in the below image.

![alt text](images/Day_23_Images/Image_3)




