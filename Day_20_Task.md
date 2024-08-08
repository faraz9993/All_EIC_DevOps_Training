# Day 20:
### In this task, I have 
### > Configured a dynamic inventory plugin to manage a growing number of web servers dynamically.
### > Ansible performance by adjusting settings such as parallel execution (forks), optimizing playbook tasks, and reducing playbook run time.
### > Included setting up verbose output.
### > Used advanced Ansible modules such as docker_container to manage containerized applications.
### > Used asynchronous for 600 in install nginx task.

### Below is my tree structure, I have used for this task:

```
.
├── aws_ec2.yaml
├── mysql
│   ├── defaults
│   │   └── main.yml
│   ├── files
│   │   └── index.html
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   │   └── main.yml
│   ├── README.md
│   ├── tasks
│   │   └── main.yml
│   ├── templates
│   │   └── nginx.conf.j2
│   ├── tests
│   │   ├── inventory
│   │   └── test.yml
│   └── vars
│       └── main.yml
└── playbook.yaml
```

### Below is my aws_ec2.yaml file which will fetch the public ip address of all the running instances in the region us-east-2:

```
plugin: amazon.aws.aws_ec2
regions:
  - us-east-2
filters:
  instance-state-name: 'running'
hostnames:
  - 'dns-name'
compose:
  ansible_host: public_ip_address
```
### Below is an image that shows the ping of the instance using aws_ec2.yaml iin the verbose mode.
![alt text](images/Day_20_Images/Image_2)

### Next I created main.yaml file which includes all the tasks to be executed on the target machine. Below is my /tasks/main.yaml file:

```
- name: Install required Python packages in virtual environment
  command: /opt/venv/bin/pip install boto3 botocore docker
  environment:
    PATH: "/opt/venv/bin:{{ ansible_env.PATH }}"

- name: Install setuptools in virtual environment
  command: /opt/venv/bin/pip install setuptools
  environment:
    PATH: "/opt/venv/bin:{{ ansible_env.PATH }}"

- name: Ensure Python3 and pip are installed
  ansible.builtin.apt:
    name:
      - python3
      - python3-pip
      - python3-venv
    state: present
    update_cache: yes

- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    async: 600

- name: Configure Nginx
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf

- name: Copy the file
  ansible.builtin.copy:
    src: index.html
    dest: /var/www/html/
  become: true

- name: Instaling python pip
  apt:
    name: python3-pip
    state: present
    update_cache: yes

- name: Create a virtual environment
  command: python3 -m venv /opt/venv

- name: Install Docker SDK in virtual environment
  command: /opt/venv/bin/pip install docker
  environment:
    PATH: "/opt/venv/bin:{{ ansible_env.PATH }}"

- name: Installing Docker
  apt:
    name: docker.io
    state: present
    update_cache: yes

- name: Pull latest nginx image
  community.docker.docker_image:
    name: nginx
    tag: latest
    source: pull

- name: Run Nginx container
  community.docker.docker_container:
    name: nginx_container
    image: nginx:latest
    state: started
    ports:
      - "80:80"
    volumes:
      - /var/www/html:/usr/share/nginx/html

- name: Stop EC2 instance
  community.aws.ec2_instance:
    state: restarted
    instance_ids:
      - "i-0a1bb810e87d517cf"
    region: us-east-2
  register: ec2
```
### Below is my vars/main.yaml file:
```
http_port: 80
server_name: 
ansible_python_interpreter: /opt/venv/bin/python
```
### Below is my nginx.conf.j2 file:

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

### Below is index.html file in files directory:
```
"This is my frontend"
```


### Below is an image that shows all the tasks ran successfully:

![alt text](images/Day_20_Images/Image_3)

### Below is an image shows the desired webpage after the execution of the above playbook.
![alt text](images/Day_20_Images/Image_4)

### Below is an image that shows the pulled image and created container after the execution of the above playbook.

![alt text](images/Day_20_Images/Image_5)


