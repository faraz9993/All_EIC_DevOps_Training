# Day 24:

### In this task, I have created and configured an S3 bucket, implemented S3 storage class, a lifecycle management for the static website, configured bucket policies and ACLs.

### First of all, I created an S3 bucket with the name "techvista-portfolio-faraz".

![alt text](images/Day_24_Images/Image_1)

### In this bucket, I have uploaded a content of the static website such as index.html and a folder with all the necassary pages.

![alt text](images/Day_24_Images/Image_2)

### Then, I implemented a storage class for the S3 bucket. I set the storage class for the "standard".

![alt text](images/Day_24_Images/Image_3)

### Furthermore, I created a life-cycle policy that transitions older versions of objects to a more cost-effective storage class (e.g., Standard to Glacier) and setting-up a policy to delete non-current versions of objects after 90 days.

![alt text](images/Day_24_Images/Image_4)

### Next, I created and attached a bucket policy that allows accessing the content with an external user, denying access from unauthorized users.

![alt text](images/Day_24_Images/Image_6)

### I am able to access my website using object url https://techvista-portfolio-faraz.s3.ap-southeast-2.amazonaws.com/carvilla-v1.0/index.html as can be seen in the below image:

![alt text](images/Day_24_Images/Image_7)


