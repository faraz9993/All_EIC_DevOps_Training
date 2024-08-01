# Day 17 Project 1: 
### In this task, I have automated the deployment and configuration of a MySQL database server on an Ubuntu EC2 instance which is hosted on AWS, and set up regular backups of database using cron job scheduling.

### I have automated the entire process using Ansible.

### First of all, I created an inventory file that will be used to connect with my target machine. My inventory file is in INI format.
```
[webservers]
web ansible_host=3.135.203.28 ansible_user=ubuntu ansible_ssh_private_key_file=/home/Ansible_Key.pem
```

### I was successfully able to ping with my target machine as can be seen in the below image.

![alt text](images/Day_17_Images/Image_1)

### Next, I built a YAML file named main.yaml which will automates the installation of MySQL, sets up the database, creates a user and configures a cron job for backups.

```
- name: Performing Day_17_Project_1
  hosts: webservers
  gather_facts: no
  become: yes
  tasks:
    - name: update cache as apt-get update
      apt:
        update_cache: yes

    - name: install mysql and its dependencies
      apt:
        name: ['mysql-server', 'mysql-client', 'python3-mysqldb', 'libmysqlclient-dev']
        state: present

    - name: enable the service
      service:
        name: mysql
        state: started
        enabled: yes
    
    - name: create a user in the database
      mysql_user:
        name: fansari
        password: fansari1234
        priv: '*.*:ALL'
        host: '%'
        state: present


# mysql -u fansari -p
# priv: '*.*:ALL' means user fansari will have all privileges (ALL) on all databases (*) and all tables (*).
# host: '%' means This specifies from which hosts the user can connect. The wildcard character % means the user can connect from any host.


    - name: changing mysql configuration file using template
      template:
        src: "/home/einfochips/All\ Extras/EIC_DevOps_Training/Ansible_Task/Day_17/templates/my.cnf.j2"
        dest: "/etc/mysql/my.cnf.j2"
      notify:
        - restart mysql
    
    - name: copy the script
      ansible.builtin.copy:
        src: "/home/einfochips/All\ Extras/EIC_DevOps_Training/Ansible_Task/Day_17/script.sh"
        dest: "/home/ubuntu/script.sh"
        mode: '777'

    - name: Set up cron job for database backup
      cron:
        name: "Daily MySQL Backup"
        minute: "*/30"
        job: /home/ubuntu/script.sh
      
  handlers:
    - name: restart mysql
      service:
        name: mysql
        state: restarted
```
### In the playbook, I had also included handlers which will restart my MySQL database service whenever a change will be made in its configuration file.

### For taking a back-up at a regular interval of every 30 minutes, I have set a cron job. The back-up will be taken using a below script.

```
#!/bin/bash
          
BACKUP_DIR=/home/ubuntu
DATE=$(date +\%F_\%T)
BACKUP_FILE="$BACKUP_DIR/mysql_backup_$DATE.sql"
MYSQL_USER=fansari
MYSQL_PASSWORD=fansari1234
MYSQL_DB=mysql
mysqldump -u $MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DB > $BACKUP_FILE
gzip $BACKUP_FILE
```
### I had first copied, this script to the target machine and then ran it with the help of cron job.

### To make the change in the configuration file of mysql, I have used a Jinja2 template. Below is that template which is name my.cnf.j2:

```
# This is a file I added using ansible. 

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

### At first, my mysql.cnf file was like this:
![alt text](images/Day_17_Images/Image_2)

### After running the playbook, it become like this:
![alt text](images/Day_17_Images/Image_5)

### I ran the playbook using below command:
```
ansible-playbook main.yaml -i inventory
```
### All, the tasks were performed successfully on the target machine as can be seen in the below image.

![alt text](images/Day_17_Images/Image_4)

### The database was succesffully created on the target machine with the user being mentioned in the playbook as can be seen in the below image:

![alt text](images/Day_17_Images/Image_6)

### The back-up was being taken successfully at a regular interval of 30 minutes using the script as can be seen in the below image:

![alt text](images/Day_17_Images/Image_7)

# Day 17 Project 2:
### In this task, I have automated the setup of a multi-tier web application stack with database and application servers using Ansible.
### I have used, MySQL database and an nginx webserver.

### First of all, I created an inventory file that will be used to connect with my target machine. My inventory file is in INI format.
```
[webservers]
web ansible_host=35.178.183.12 ansible_user=ubuntu ansible_ssh_private_key_file=/home/Ansible_Key.pem
```

### I was successfully able to ping with my target machine as can be seen in the below image.

![alt text](images/Day_17_Images/Image_3)

### Next, I built a YAML file named main2.yaml which will automates the installation of mysql, sets up the database, creates a user and configures a cron job for backups.

```
- name: Performing Day_17_Project_2
  hosts: webservers
  gather_facts: no
  become: yes
  tasks:
    - name: update cache as apt-get update
      apt:
        update_cache: yes

    - name: install mysql and its dependencies
      apt:
        name: ['mysql-server', 'mysql-client', 'python3-mysqldb', 'libmysqlclient-dev']
        state: present

    - name: install nginx
      apt:
        name: nginx
        state: present

    - name: enable the mysql service
      service:
        name: mysql
        state: started
        enabled: yes

    - name: enable the nginx service
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: create a user in the database
      mysql_user:
        name: fansari
        password: fansari1234
        priv: '*.*:ALL'
        host: '%'
        state: present

# priv: '*.*:ALL' means user fansari will have all privileges (ALL) on all databases (*) and all tables (*).
# host: '%' means This specifies from which hosts the user can connect. The wildcard character % means the user can connect from any host.

    - name: changing the nginx config file using template
      template:
        src: "/home/einfochips/All\ Extras/EIC_DevOps_Training/Ansible_Task/Day_17/index.html"
        dest: "/var/www/html/index.html"
      notify:
        - restart nginx

    - name: copy the script
      ansible.builtin.copy:
        src: "/home/einfochips/All\ Extras/EIC_DevOps_Training/Ansible_Task/Day_17/script.sh"
        dest: "/home/ubuntu/script.sh"
        mode: '777'
      
  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted

#mysql -u fansari -p
```
### Below is my index.html file which I copied to the target machine:
```
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple MySQL Connection</title>
</head>
<body>
    <h1>This is my nginx webpage</h1>
    <form action="process.php" method="POST">
        <input type="submit" value="Check Database Connection">
    </form>
</body>
</html>


```
### Below is my nginx configuration file which I created using a template which is taking a variable from the /var/main.yaml file:

```
server {
    listen {{ http_port }};
    server_name {{ ip_address }};

    root /var/www/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}

```

### /var/main.yaml:

```
file_name: 'index.html'
http_port: 80 
ip_address: 3.129.67.224
database_host: 'localhost'
database_user: 'fansari'
database_password: 'fansari1234'
database_name: 'mysql'
```
### Below is my process.php file which gives an output that web server is successfully connected with the database.

```
<?php
$servername = "localhost";
$username = "fansari";
$password = "fansari1234";
$dbname = "mysql";

// Create connection

$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connection with the database is successfull";
?>

```

### All the tasks were successfully executed on the target machine as can be seen in the below image.


![alt text](images/Day_17_Images/Image_11)

### Below is my frontend application:

![alt text](images/Day_17_Images/Image_8)

### Below page shows my successful connection with the database as per my process.php file.

![alt text](images/Day_17_Images/Image_10)