# Day 3 Task: Docker

#### In this task I had learned several basic Docker commands and also also created a frontend webpage, backend and database using Docker out of which only fronend is publically accessible and not backend and database.

First of all, I had pulled the image of nginx using below command:

```
docker pull nginx
```

Then later ran a container using this image where I had exposed the port 80 of the container to the port 80 of the host.

```
docker run --name my-nginx -d -p 8080:80 nginx
```
In the above command, 

```
--name my-nginx: Assigns a name to the container.
-d: Runs the container in detached mode.
-p 8080:80: Maps port 8080 on your host to port 80 in the container.
```

You can check the details of all the currently running containers by the below command:
```
docker ps
```

As the container was running, when I hit the http://localhost:8080 in my local host browser, I was able to get the nginx default webpage as shown in the below image:

![alt text](/images/Day_3_Images/Image_7)

Later on, I went into the container using below command:
```
docker exec -it my-nginx /bin/bash
```

and made the changes in the HTML page:

```
echo "<html><body><h1>Hello from Docker</h1></body></html>" > /usr/share/nginx/html/index.html
```

After making this change I was able to get this text "Hello from Docker" on my webpage as shown in the below image:

![alt text](/images/Day_3_Images/Image_9)

Next, I used this container to create a new image out of it using below command:

```
docker commit my-nginx custom-nginx
```

In the above command "my-nginx" is the name of the container and the "cutom-nginx" is the name we gave to the image that will be created out of it.

Next, I created a Dockerfile to Build and Deploy a Web Application.

first of all, I created a directory and inside that directory I created an index.html file with the below content inside it:

```
<!DOCTYPE html>
<html>
<body>
    <h1>Hello from My Web App!</h1>
</body>
</html>
```

Secondly, in the same directory I made another file with the name "Dockerfile" with the below content inside it:

```
# Use the official Nginx base image
FROM nginx:latest

# Copy the custom HTML file to the appropriate location
COPY index.html /usr/share/nginx/html/

# Expose port 80
EXPOSE 80
```
Now, to build the docker image I ran the below given command:

```
docker build -t my-webapp-image .
```

Here, "my-webapp-image" is the name I gave to the image that will be created and . is for copying all the files in the directory to within the image.

Then, as the image is built, I ran a Container from the built Image using the below command:

```
docker run --name my-webapp-container -d -p 8082:80 my-webapp-image
```

When I searched "http://localhost:8082" I was able to get my desired webpage as shown in the below image.

![alt text](images/Day_3_Images/Image_13)


-----

## building a full-stack application using Docker.

First of all, I created a directory with the name "fullstack-docker-app" and inside that directory created three sub-dircetories "frontend", "backend" and "database".

As was going to built a full-stack project, I had created a network and the volume as well using below commands:

```
docker network create fullstack-network
```
```
docker volume create pgdata
```
### Creating a Database 
Now, to create a databse, I created a Dockerfile

```
FROM postgres:latest
ENV POSTGRES_USER=user
ENV POSTGRES_PASSWORD=password
ENV POSTGRES_DB=mydatabase
```
I created a postgreSQL image using command:
```
docker build -t my-postgres-db .
```
and ran the postgresql container using the netwok and volume I created earlier.
```
docker run --name postgres-container --network fullstack-network -v pgdata:/var/lib/postgresql/data -d my-postgres-db
```
### Creating a backend

As the backend code is of Node.js, I went into the backend dirctory and ran the command:

```
npm init -y
```

This command will initializes a new Node.js project by creating a package.json file with default values where -y flag automatically answers "yes" to all the prompts.

Then I run the below command to install Express and pg (PostgreSQL) which is a client for Node.js.

```
npm install express pg
```

Below is the backend code which will be saved in the file named "index.js"

```
const express = require('express');
const { Pool } = require('pg');
const app = express();
const port = 3000;

const pool = new Pool({
    user: 'user',
    host: 'postgres-container',
    database: 'mydatabase',
    password: 'password',
    port: 5432,
});

app.get('/', (req, res) => {
    res.send('Hello from Node.js and Docker!');
});

app.get('/data', async (req, res) => {
    const client = await pool.connect();
    const result = await client.query('SELECT NOW()');
    client.release();
    res.send(result.rows);
});

app.listen(port, () => {
    console.log(`App running on http://localhost:${port}`);
});
```

Then, I created a Dockerfile for the backend:
```
FROM node:latest

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000
CMD ["node", "index.js"]
```

I built the docker image of this Dockerfile using the command:

```
docker build -t my-node-app .
```

and ran the container of that image with the same network and volumen we created earlier:

```
docker run --name backend-container --network fullstack-network -d my-node-app
```

### Creating a frontend

The next step is to create a front-end container.

Below is the code in HTML which show a weboage for the frontend:

```
<!DOCTYPE html>
<html>
<body>
    <h1>Hello from Nginx and Docker!</h1>
    <p>This is a simple static front-end served by Nginx.</p>
    <a href="http://localhost:3000/data">Fetch Data from Backend</a>
</body>
</html>
```

Below is the Dockerfile for the frontend:

```
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
```

THe command for creating the image of this Dockerfile:

```
docker build -t my-nginx-app .
```

Afterwards, I ran the container using below command of the image that is created

```
docker run --name frontend-container --network fullstack-network -p 8080:80 -d my-nginx-app
```

This is the complete project. Now next step that comes is to verify all the things.
The first thing to verify is the connection between the Bakcend and the Database.

For that we need to get into the backend container using exec command and run the below given commands:

```
docker exec -it backend-container /bin/bash
apt-get update && apt-get install -y postgresql-client
psql -h postgres-container -U user -d mydatabase -c "SELECT NOW();"
```
If the connection between backend and postgresql would be successful it will show you the details of the database as shown in the below image:

![alt text](images/Day_3_Images/Image_22)

Then when we will hit the "http://localhost:8080" in the browser of the localhost it would show the below webpage:

![alt text](images/Day_3_Images/Image_24)

The link given on the webpage is for connecting with the backend so, when you will click on it it should not take you to the backend which is a basic requirement as the backend of the server should not be publicly accessible. 

