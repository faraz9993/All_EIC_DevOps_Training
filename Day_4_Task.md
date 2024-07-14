# In this Task I have learned about Docker Swam

Docker Swarm is a container orchestration tool that allows you to manage a cluster of Docker nodes as a single logical system.
It provides several benefits, such as scalability, high availability, load balancing, and simplified deployment. Here are some use cases and examples of how Docker Swarm can be utilized.

For this task, I needed two nodes. One of the nodes will be manager node and another node will be worker node.

For that, I created two ubuntu server using AWS EC2 instance. 

In security group I  allowed ALL TCP traffic so both the nodes can easily connect with each other using token.

In bothe the nodes I install docker swarm using below commands:

```
apt-get update && apt-get install docker.io -y
```

Afterwards, I initialized docker swarm using command:

```
docker swarm init --advertise-addr <MANAGER-IP>
```

When I ran the above command, the token was generated. Using that token we can connect the worker node with this manager-node.

```
docker swarm join --token <WORKER-TOKEN> <MANAGER-IP>:2377
```


![alt text](images/Day_4_Images/Image_2)


I deployed a nginx web-application using below command:

```
docker service create --name webapp --replicas 3 -p 80:80 nginx
```

To check service status:

```
docker service ls
```


![alt text](images/Day_4_Images/Image_8)


To deploy Jenkins using Docker swarm:

```
docker swarm init
docker service create --name jenkins --replicas 1 -p 8080:8080 jenkins/jenkins
```
To deploy a Service with Load Balancing:

```
docker service create --name myservice --replicas 5 -p 8080:80 nginx
```

To scale the Service:

```
docker service scale myservice=10
```


![ alt text](images/Day_4_Images/Image_10)


To deploy Microservices:

```
docker service create --name service1 --replicas 3 -p 5000:5000 my_microservice1
docker service create --name service2 --replicas 2 -p 5001:5001 my_microservice2
```

## Docker Logs

```
docker logs <container_name_or_id>
```

![alt text](images/Day_4_Images/Image_11)

Following Logs in Real-Time

```
docker logs -f my_container
```

![alt text](images/Day_4_Images/Image_12)

Showing the Last 10 Lines of Logs
```
docker logs --tail 10 my_container
```
![alt text](images/Day_4_Images/Image_13)


Showing Logs with Timestamps
```
docker logs -t my_container
```

![alt text](images/Day_4_Images/Image_14)

Showing Logs Since a Specific Time
```
docker logs --since "2023-07-11T15:00:00" my_container
```
![alt text](images/Day_4_Images/Image_15)


Combining Options
```
docker logs -f --tail 10 --since "10m" my_container
```
![alt text](images/Day_4_Images/Image_16)

