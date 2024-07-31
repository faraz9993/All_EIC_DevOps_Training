### Day 17 Project 1: In this task, I have automated the deployment and configuration of a PostgreSQL database server on an Ubuntu EC2 instance which is hosted on AWS, and set up regular backups using cron job scheduling.

### I have performed this entire task using Ansible.

### First of all, I created an inventory file that will be used to connect with my target machine. My inventory file is in INI format.
```
[webservers]
web ansible_host=3.135.203.28 ansible_user=ubuntu ansible_ssh_private_key_file=/home/Ansible_Key.pem
```

### I was successfully able to ping with my target machine as can be seen in the below image.

![alt text](images/Day_17_Images/Image_1)

### Next, I built a YAML file named main.yaml which will automates the installation of PostgreSQL, sets up the database, creates a user and configures a cron job for backups.

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
### In the playbook, I had also included handlers which will retart my MySQL database service whenever a change will be made in its configuration file.

### For taking a back-up at a regular interval of every 30 minutes, I have set a cron job as well. The back-up will be taken using a below script.

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

### Day 17 Project 2: In this task, I have automated the setup of a multi-tier web application stack with database and application servers using Ansible.
### I have used, MySQL database and an nginx webserver.

### First of all, I created an inventory file that will be used to connect with my target machine. My inventory file is in INI format.
```
[webservers]
web ansible_host=35.178.183.12 ansible_user=ubuntu ansible_ssh_private_key_file=/home/Ansible_Key.pem
```

### I was successfully able to ping with my target machine as can be seen in the below image.

![alt text](images/Day_17_Images/Image_3)

### Next, I built a YAML file named main2.yaml which will automates the installation of PostgreSQL, sets up the database, creates a user and configures a cron job for backups.

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
    - name: restart mysql
      service:
        name: mysql
        state: restarted

    - name: restart nginx
      service:
        name: nginx
        state: restarted

#mysql -u fansari -p
```
### Below is my index.html file which I copied to the target machine:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>User Input Form</title>
</head>
<body>
    <h2>User Information</h2>
    <form action="/submit" method="post">
        <label for="name">Name:</label>
        <input type="text" id="name" name="name" required><br><br>
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required><br><br>
        <button type="submit">Submit</button>
    </form>
</body>
</html>

```
### Below is my nginx configuration file which I created using a template which is taking a variable from the /var/main.yaml file:

```
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       {{ http_port }};
        server_name  localhost;

        location / {
            root   /var/www/html;
            index  {{ file_name }};
        }
    }
}
```

### /var/main.yaml:

```
file_name: 'index.html'
http_port: 80 
```

