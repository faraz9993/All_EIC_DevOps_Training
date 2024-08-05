### In this task I have deployed a three-tier web application (frontend, backend, and database) using Ansible roles. The frontend is an Nginx web server, the backend is a Node.js application and the database is a MySQL server. I have used Ansible Galaxy roles where applicable and defined appropriate role dependencies. The deployment has automated to ensure that all components are configured correctly and can communicate with each other.

### FIrst od all, I created an inventory file:

```
[webservers]
web ansible_host=3.142.250.171 ansible_user=ubuntu ansible_ssh_private_key_file=/home/einfochips/Downloads/ansible-worker.pem
```

### Next, I created 3 roles using below commands:

```
ansible-galaxy init frontend
ansible-galaxy init backend
ansible-galaxy init database
```

### Below, is directory structure after creating the roles:

```
.
├── ansible.cfg
├── backend
│   ├── defaults
│   │   └── main.yml
│   ├── files
│   │   └── app
│   │       ├── app.js
│   │       └── package.json
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   │   └── main.yml
│   ├── README.md
│   ├── tasks
│   │   └── main.yml
│   ├── templates
│   │   └── app.js.j2
│   ├── tests
│   │   ├── inventory
│   │   └── test.yml
│   └── vars
│       └── main.yml
├── database
│   ├── defaults
│   │   └── main.yml
│   ├── files
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   │   └── main.yml
│   ├── README.md
│   ├── tasks
│   │   └── main.yml
│   ├── templates
│   │   └── my.cnf.j2
│   ├── tests
│   │   ├── inventory
│   │   └── test.yml
│   └── vars
│       └── main.yml
├── frontend
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
├── inventory
└── playbook.yaml

28 directories, 33 files
```

### Below is my frontend/tasks/main.yaml file:
```
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present

- name: Configure Nginx
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf

- name: Start Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: yes
```

### frontend/files/index.html:

```
echo "This is my frontend"
```

### frontend/templates/nginx.conf.j2:
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

### frontend/vars/main.yaml:
```
http_port: 80
server_name: nginx_server
```

### below is my backend.tasks/main.yaml file:
```
---
- block:
    - name: Install Node.js
      ansible.builtin.apt:
        name: nodejs
        state: present
      become: yes
    
    - name: Install npm (Node.js package manager)
      ansible.builtin.apt:
        name: npm
        state: present
      become: yes
    
    - name: Create application directory
      ansible.builtin.file:
        path: /var/www/app
        state: directory
        mode: '0755'
      become: yes

    - name: Deploy Node.js application
      ansible.builtin.copy:
        src: app/
        dest: /var/www/app/
      become: yes

    - name: Install npm dependencies
      ansible.builtin.npm:
        path: /var/www/app
        production: yes
      become: yes

    - name: Create systemd service file for Node.js app
      ansible.builtin.copy:
        content: |
          [Unit]
          Description=Node.js Application

          [Service]
          ExecStart=/usr/bin/node /var/www/app/app.js
          Restart=always
          User=nobody
          Group=nogroup
          Environment=PATH=/usr/bin:/usr/local/bin
          Environment=NODE_ENV=production
          WorkingDirectory=/var/www/app

          [Install]
          WantedBy=multi-user.target
        dest: /etc/systemd/system/nodeapp.service
      become: yes

    - name: Start and enable Node.js app service
      ansible.builtin.systemd:
        name: nodeapp
        enabled: yes
        state: started
      become: yes
```
### backend/files/app/app.js:
```
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello, World!');
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});

```
### backend/files/app/package.json:

```
{
    "name": "myapp",
    "version": "1.0.0",
    "description": "A simple Node.js application",
    "main": "app.js",
    "scripts": {
      "start": "node app.js"
    },
    "dependencies": {
      "express": "^4.17.1"
    }
  }
  
```

### backend/meta/main.yaml:
```
dependencies:
  - role: frontend
```

### backend/templates/app.js.j2:
```
const http = require('http');
const hostname = '0.0.0.0';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```
### below is my database/tasks/main.yaml file:

```
---
- block:
    - name: Install MySQL server
      apt:
        name: mysql-server
        state: present
      become: yes

    - name: Install required Python packages for MySQL
      apt:
        name: "{{ item }}"
        state: present
      with_items:
        - python3-pymysql
        - python3-mysqldb
      become: yes

    - name: Configure MySQL
      template:
        src: my.cnf.j2
        dest: /etc/mysql/my.cnf
      become: yes

    - name: Start MySQL server
      service:
        name: mysql
        state: started
        enabled: yes
      become: yes

```
### database/meta/main.yaml:

```
dependencies:
  - role: frontend
  - role: backend
```
### database/templates/my.cnf.j2:
```
[mysqld]
bind-address = 0.0.0.0

```