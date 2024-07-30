# Day 16: In this task, I have created an instance on AWS and used it as a target machine to perform several Ansible tasks.

### Below is my inventory file which I used to ping myh target machine.

### inventory.cfg
```
[webservers]
web ansible_host=3.16.78.37 ansible_user=ubuntu ansible_ssh_private_key_file=/home/einfochips/Downloads/ansible-worker.pem
```
### In the inventory file, I mentioned a few details which were needed for pinging the mahcine such as the IP address as ansible_host, username as ansible_user and the location of pem file of the target machine.

### I also changed the permission of the pem file to read only for the owner using below command.

```
chmod 400 /home/einfochips/Downloads/ansible-worker.pem
```
### I ping my target machine using the below command:
```
ansible all -i inventory.cfg -m ping
```
![alt text](images/Day_16_Images/Image_1)

### Once, I was successfully able to ping my machine, I ran a few ad-hoc commands such as:

### 1. Disk usage:
```
ansible all -i inventory.cfg -a "du -h"
```
![alt text](images/Day_16_Images/Image_5)

### 2. Restarting the service:
```
ansible all -i inventory.cfg -b -m service -a "name=nginx state=restarted"
```
![alt text](images/Day_16_Images/Image_11)

### 3. Updating the packages:
```
ansible all -i inventory.cfg -b -m apt -a "update_cache=yes"
```
![alt text](images/Day_16_Images/Image_3)

### Static Inventory is typically a file where the hosts are listed along with the groups they belong to.
```
[development]
dev1.example.com
dev2.example.com

[staging]
stage1.example.com
stage2.example.com
```
### Dynamic inventory is a script or a plugin that fetches the inventory from external sources like cloud providers, databases or other services.

### Next, I created a playbook that will install a nginx webserver on my target machine. Not only this, I also created a file, modified the content inside it and deleted it. Below is that main.yaml file:

```
  - name: Project 4 tasks
    hosts: webservers
    gather_facts: no
    become: yes
    tasks:
    - name: installing nginx
      apt:
        name: nginx
        state: present
    - name: creating a file
      file:
          path: /home/faraz.txt
          state: touch
    - name: modifying the content inside the file
      lineinfile:
        path: /home/faraz.txt
        line: "Hello There!"
        create: yes
    - name: Delete a file
      file:
        path: /home/faraz.txt
        state: absent
```

### I ran the above playbook using below command.
```
ansible-playbook main.yaml -i inventory.cfg
```

### All the tasks were performed successfully as can be seen the in the below image.

![alt text](images/Day_16_Images/Image_8)

### Furthermore, I implemented an error handling strategies using modules like Block and Rescue.
### I used two links that will give me a list of all the engineering colleges in Ahmedabad city. In case, if the first link fails, it will download the second link which is to rescue the deployment failure. 

```
- name: download college_list
      block:
        - get_url:
            url: http://be.jacpcldce.ac.in/govt/pdf/List%20of%20Institute%20with%20MQ_NRI%20and%20Entrance%20exam%20Bifurcation.pdf
            dest: /home/college_list
        - debug: msg="College list downloaded"
      rescue:
        - debug: msg="College list not available"
        - get_url:
            url: https://www.aicte-india.org/sites/default/files/Engineering.pdf
            dest: /home/college_list
```
![alt text](images/Day_16_Images/Image_10)

