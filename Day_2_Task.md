# Day 2 Task: Git
In this task, I have created a git repository and performed several git commands.

For this task, I have used git version 2.25.1. You can check the git version using below command:

```
git --version
```

I created a git repository on Git and clonned it into my machine so I can use it using CLI.

To clone I ran the below command:

```
git clone <https:>
```

Afterwards, I ran the git init command in the directory where I have clonned the git repo which sets up the necessary files and structures for Git to track changes in that directory.

![alt text](/images/Day_2_Images/image_1)

I had pushed the static website that I had created in task 1 on this git repo. 

For that I copied the required necessary files and directories in the directory where I had initialized the git and ran below three commands to push these files:

```
git add .
git commit -m "Adding Day 1 Task"
git push
```

![alt text](/images/Day_2_Images/image_2)

The . in git add . will add all the files in the present directory to the staging area.

Next, I created another branch named "feature" using the below command:

```
git branch feature
```
![alt text](/images/Day_2_Images/image_3)


Afterwards, I changed the branch using command:

```
git checkout feature
```
and pushed a sample code into it.

![alt text](/images/Day_2_Images/image_4)


![alt text](/images/Day_2_Images/image_5)

Later on, I checkedout to the main branch and merged the feature branch into it.

![alt text](/images/Day_2_Images/image_6)

If you have done a manual changes in the git repo using GUI and you want to pull it into your git repo in the local, you can run the below command:

```
git pull origin main
git pull origin feature
```
![alt text](/images/Day_2_Images/image_7)

IN git tags are used to mark release points (e.g., v1.0, v2.1) in your project's development. 

To create a tag in the git:

```
git tag v1.0.0
```

![alt text](/images/Day_2_Images/image_8)

and then you have to push it using the command

```
git push origin v1.0.0
```
The next is git stash. Git stash is used to perform half commit.

git stash acts as a version control tool and lets developers work on other activities or switch branches in Git without having to discard or commit changes that aren't ready.

Developers can simply stash the changes in their working directory and index state and work on them later.

![alt text](/images/Day_2_Images/image_14)
![alt text](/images/Day_2_Images/image_15)


Later on, I perform git cherry-picking.

git cherry-pick Cherry picking is the act of picking a commit from a branch and applying it to another. 

git cherry-pick can be useful for undoing changes. For example, say a commit is accidently made to the wrong branch. 

You can switch to the correct branch and cherry-pick the commit to where it should belong.

```
git cherry-pick <commit_id>
```

![alt text](/images/Day_2_Images/image_12)
![alt text](/images/Day_2_Images/image_13)


Afterwards, comes git rebase. git rebase is a Git feature which is used to integrate changes from one branch into another. 

It does this by moving or combining a series of commits to a new base commit. 

![alt text](/images/Day_2_Images/image_11)