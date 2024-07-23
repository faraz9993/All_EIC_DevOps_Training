# Task:11 Jenkins Pipeline

### First of all, I created the private Git repo in which I pushed the code provided by the instructor. Then I went to the Jenkins where I created two projects.

### 1. Demo_Dev
### 2. Demo_Test

### In Demo Dev, I selected Git as a Source Code Management. In it, I entered the URL of my private repo and entered the token as a username and password.

### Secondly, I selected the main branch. 

### Later on, I went to the Build Steps slected Maven Version. Also, in Dashboard > Manage Jenkins > Maven Installation, I added a maven version.

![alt text](images/Day_11_Images/Image_2)

### Furthermore, In build steps, I selected the Maven version and in goals I wrote compile as I wanted to compile the code. I applied and saved the code.

### Now, I created the second project with the same configuration just in place of "compile" at Goals I wrote "test".

![alt text](images/Day_11_Images/Image_4)

### Afterwards, I created a post-build action and entered the "demo_test" and selected trigger even if the build is unstable.

![alt text](images/Day_11_Images/Image_5)


### Then On the dashboard I created a new view as shown in below images: 

![alt text](images/Day_11_Images/Image_6)
