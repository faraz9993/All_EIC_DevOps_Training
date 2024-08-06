# Day 19:
### In this task, I have 
### Set up an AWS EC2 instance as a worker node.
### Implement Ansible playbooks and roles following best practices.
### Use version control to manage Ansible codebase.
### Document Ansible roles and playbooks.
### Break down deployment tasks into reusable roles.
### Write reusable and maintainable Ansible code.
### Use dynamic inventory scripts to manage AWS EC2 instances.
### Deploy a web application on the EC2 instance.

### So, first of all, I created a role for nginx installation on the target machines using below command:
```
ansible-galaxy init nginx
```
### Below is my directory structure.

.
├── nginx
│   ├── defaults
│   │   └── main.yml
│   ├── files
│   │   └── index.html
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   │   └── main.yml
│   ├── README.md
│   ├── tasks
│   │   └── main.yml
│   ├── templates
│   │   └── nginx.conf.j2
│   ├── tests
│   │   ├── inventory
│   │   └── test.yml
│   └── vars
│       └── main.yml
└── playbook.yaml

### Below is my playbook.yaml:
```
---
- name: Deploy fronend application using dynamic inventory
  hosts: all
  become: yes

  roles:
    - role: mysql
```

### This is my tasks/main.yml:
```
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present

- name: Configure Nginx
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf

- name: Copy the file
  ansible.builtin.copy:
    src: index.html
    dest: /var/www/html/
  become: true

- name: Start Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: yes
```
### Below is my files/index.html:
```
"This is my frontend"
```
### Below is my templates/nginx.conf.j2:
```
server {
    listen {{ http_port }};
    server_name {{ server_name }};

    location / {
        proxy_pass http://backend.dev.local:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
### Below is my vars/main.yaml:
```
http_port: 80
server_name: nginx_server
```

### As I am creating a dynamic inventory, I created a python script which will fetch the public ip address of the target machines. Below is my python script:
```
#!/usr/bin/env python3

import boto3
import json
import sys

def get_ec2_instances():
    print(f"Using Python: {sys.executable}", file=sys.stderr)
    
    session = boto3.Session(region_name="us-east-2")
    ec2 = session.client("ec2")

    response = ec2.describe_instances()

    inventory = {
        "_meta": {
            "hostvars": {}
        },
        "nginx_servers": {
            "hosts": []
        }
    }

    ssh_key_file = '/home/einfochips/Downloads/Faraz_Key_1.pem'
    ssh_user = 'ubuntu'

    for reservation in response["Reservations"]:
        for instance in reservation["Instances"]:
            if instance["State"]["Name"] == "running":
                public_ip = instance.get("PublicIpAddress")
                if public_ip:
                    inventory["nginx_servers"]["hosts"].append(public_ip)
                    inventory["_meta"]["hostvars"][public_ip] = {
                        "ansible_host": public_ip,
                        "ansible_ssh_private_key_file": ssh_key_file,
                        "ansible_user": ssh_user
                    }
                    print(f"Added {public_ip} with key {ssh_key_file} and user {ssh_user}", file=sys.stderr)

    return inventory

if __name__ == "__main__":
    print(json.dumps(get_ec2_instances(), indent=4))

```
### When I ran the python script using command 'python3 ec2_inventory.py' it gives me the below output:
```
Using Python: /usr/bin/python3
Added 18.222.227.133 with key /home/einfochips/Downloads/Faraz_Key_1.pem and user ubuntu
{
    "_meta": {
        "hostvars": {
            "18.222.227.133": {
                "ansible_host": "18.222.227.133",
                "ansible_ssh_private_key_file": "/home/einfochips/Downloads/Faraz_Key_1.pem",
                "ansible_user": "ubuntu"
            }
        }
    },
    "nginx_servers": {
        "hosts": [
            "18.222.227.133"
        ]
    }
}
```
### This output proves that my python script is working completely fine.
### So, now this is the time to run the playbook.
```
ansible-playbook playbook.yaml 
```
### As shown in the below images, when I ran the above command, all the tasks were performed succesfully and when I hit the public ip address in the browser,I got the desired webpage as well.

![alt text](images/Day_19_Images/Image_1)
![alt text](images/Day_19_Images/Image_2)

### I had pushed the entire role to the github as a version control system as can be seen in the below image:

![alt text](images/Day_19_Images/Image_3)