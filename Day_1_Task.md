# Day 1 Task
- In this task, I have performed several basic linux commands.

- First of all, I have created a file named server_config.txt using nano command and added a context inside it.

```
nano server_config.txt 
```
and pasted the below content inside it.

```
Server Name: WebServer01
IP Address: 192.168.1.100
OS: Ubuntu 20.04
```
I saved the file using ctrl+s command and later using ctrl+x command.

-----


After that I used vim command to add the below text inside that file.

```
vim server_config.txt
```
```
Configuration Complete: Yes
```

![alt text](/images/image-2.png)

We have to press escape key then write wq! whre w is for save q is for quit and ! is for forceful execution.

```
wq!
```
-----
- Next, I have used to the cat command to see the context within the server_config.txt file I created using nano command.

```
cat server_config.txt
```
![alt text](/images/image-1.png)
-----


- Next, I added a user named developer using a command adduser.

```
sudo adduser developer
```
![alt text](/images/image-3.png)

-----
- I have also added a group named devteam using the command groupadd.

```
sudo groupadd devteam
```
![alt text](/images/image-4.png)

-----
- In the below image, I have removed a user named developer from the group called devteam using command gpasswd -d.

```
sudo gpasswd -d developer devteam
```
![alt text](/images/image-5.png)

------
- Further, I have used ls -l command to see the long listing of the file inside the present directory as it shows the permissions and ownership of the files and directories.

```
ls -l server_config.txt
```
![alt text](/images/image-6.png)

------

- In the below image, I haved used chmod command to change the permission of the file named server_config.txt.

```
chmod 644 server_config.txt
```
![alt text](/images/image-7.png)

----
- Later, I have downloaded the apache2.service using apt install command.

```
sudo apt install apache2
```
or we can also use

```
sudo apt-get install apache2
```
![alt text](/images/image-8.png)

-----------
- In the below image, I have used systemctl command to change the status of the apache2 service.

```
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```
![alt text](/images/image-9.png)

------

- Furthermore, I have run the ps aux command to see all the running processes.

```
ps aux
```
![alt text](/images/image-10.png)


-----
- Next, I have ran the top command and used kill command to kill one of the running processes.

```
top
```
![alt text](/images/image-11.png)


-----

- Furthermore, I have used the nice command which lets me run a command at a priority lower than the command's normal priority.
```
nice -n 10 sleep 100 &
```

& will make the command run in the back ground.

![alt text](/images/image-12.png)

-----

# Creation of Static website on Apache Server

- As shown in the below image, I have created a directory named mystaticwebsite at the default path of the apache server which is /var/www/html/ .

![alt text](/images/image-13.png)


- Furthermore, I have also created three files in it and added content inside it using vim command.
- Not only this, I have also added a logo.png image which will appear on my website.

![alt text](/images/image-14.png)

The content of styles.css is:
```
body {
  font-family: Arial, sans-serif;
  background-color: #f0f0f0;
  text-align: center;
  margin: 0;
  padding: 20px;
}

h1 {
  color: #333;
}

```

The content of script.js is:
```
document.addEventListener('DOMContentLoaded', function() {
  console.log('Hello, World!');
});
```

The content of index.html is:
```
<body>
<center>
    <h1>Welcome to My Static Website</h1>
    <img src="logo.png" alt="Logo" height="400" >
    <p>This is a simple static website using Apache2.</p>
    <script src="script.js"></script>
</center>
</body>

```
-----
- Next, I have disabled original apache2 configuration file using the command sudo a2dissite 000-default.conf and enabled the configuration file for my staticwebsite using the command sudo a2ensite mystaticwebsite.conf and then reloaded the service using systemctl command.

![alt text](/images/image-15.png)
------

- Now I got the IP address of my local system using the command ifconfig.
- When I hit that IP into tge browser, I was able to get my desired webpage as shown in below image.

![alt text](/images/image-16.png)
---