# Assesment_Day:
### In this task, I have done a set up of microservices-based application. The application will be containerized using Docker and orchestrated using Kubernetes.

### First of all, I created 3 services one for application frontend, one for Order Processing and one for Product Catalogue. All the services were created and interconnected using HTML.

### Below is my index.html file for the Frontend:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Front-end Microservice</title>
    <style>
        body { font-family: Arial, sans-serif; }
        .container {
            text-align: center;
            margin-top: 50px;
        }
        .navigation {
            margin-top: 20px;
        }
        button {
            margin: 5px;
            padding: 10px 20px;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Front-end Microservice</h1>
        <p>Welcome to the front-end service!</p>
        <div class="navigation">
            <button onclick="window.location.href='http://192.168.49.2:30002/index.html'">Product Catalog</button>
            <button onclick="window.location.href='http://192.168.49.2:30003/index.html'">Order Processing</button>
        </div>
    </div>
</body>
</html>
```
### Below is my index.html file for the Order Processing:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Order Processing Microservice</title>
    <style>
        body { font-family: Arial, sans-serif; }
        .container {
            text-align: center;
            margin-top: 50px;
        }
        .navigation {
            margin-top: 20px;
        }
        .orders {
            margin-top: 20px;
        }
        button {
            margin: 5px;
            padding: 10px 20px;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Order Processing Microservice</h1>
        <button onclick="createOrder()">Create Order</button>
        <div class="orders"></div>
        <div class="navigation">
            <button onclick="window.location.href='http://192.168.49.2:30001/index.html'">Front-end</button>
            <button onclick="window.location.href='http://192.168.49.2:30002/index.html'">Product Catalog</button>
        </div>
    </div>

    <script>
        const orders = [];

        function createOrder() {
            const orderId = orders.length + 1;
            const order = { id: orderId, products: ['Product 1', 'Product 2', 'Product 3'] };
            orders.push(order);
            showOrders();
        }

        function showOrders() {
            const ordersDiv = document.querySelector('.orders');
            ordersDiv.innerHTML = '';
            orders.forEach(order => {
                const orderDiv = document.createElement('div');
                orderDiv.textContent = `Order ${order.id} - ${order.products.length} products`;
                ordersDiv.appendChild(orderDiv);
            });
        }
    </script>
</body>
</html>
```

### ### Below is my index.html file for the Product Catalogue :
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Product Catalog Microservice</title>
    <style>
        body { font-family: Arial, sans-serif; }
        .container {
            text-align: center;
            margin-top: 50px;
        }
        .navigation {
            margin-top: 20px;
        }
        .products {
            margin-top: 20px;
        }
        button {
            margin: 5px;
            padding: 10px 20px;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Product Catalog Microservice</h1>
        <button onclick="showProducts()">Show Products</button>
        <div class="products"></div>
        <div class="navigation">
            <button onclick="window.location.href='http://192.168.49.2:30001/index.html'">Front-end</button>
            <button onclick="window.location.href='http://192.168.49.2:30003/index.html'">Order Processing</button>
        </div>
    </div>

    <script>
        const products = [
            { id: 1, name: 'Product 1', price: 100 },
            { id: 2, name: 'Product 2', price: 200 },
            { id: 3, name: 'Product 3', price: 300 }
        ];

        function showProducts() {
            const productsDiv = document.querySelector('.products');
            productsDiv.innerHTML = '';
            products.forEach(product => {
                const productDiv = document.createElement('div');
                productDiv.textContent = `${product.name} - $${product.price}`;
                productsDiv.appendChild(productDiv);
            });
        }
    </script>
</body>
</html>
```

#### I created the Dockerfile for all three services:
### I used the same Dockerfile for the Frontend, Order Processing and Product Catalogue :

```
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
### After that I created a docker images out of these files and pushed them to the docker hub.
### Below are the commands I used for building the docker images, creating the tag and pushing them to the docker hub.

```
docker build -t front-end .
docker build -t product-catalog .
docker build -t order-processing .


docker tag front-end fansari9993/test9:front-end
docker tag product-catalog fansari9993/test9:product-catalog
docker tag order-processing fansari9993/test9:order-processing


docker push fansari9993/test9:front-end
docker push fansari9993/test9:product-catalog
docker push fansari9993/test9:order-processing
```

![alt text](images/Assessment_Day/Image_12)

### After pushing the images to the docker hub I created kubernetes manifest files for each microservice.

### Below is my frontend.yaml file for frontend deployment and service:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: front-end
spec:
  replicas: 2
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
      labels:
        app: front-end
    spec:
      containers:
      - name: front-end
        image: fansari9993/test9:front-end
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "500m"
          limits:
            cpu: "1"
---
apiVersion: v1
kind: Service
metadata:
  name: front-end-service
spec:
  selector:
    app: front-end
  ports:
  - protocol: TCP
    port: 8091
    targetPort: 80
    nodePort: 30001
  type: NodePort
```
### Below is my product-catalog.yaml file for deployment and service of Product Catalog:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-catalog
spec:
  replicas: 2
  selector:
    matchLabels:
      app: product-catalog
  template:
    metadata:
      labels:
        app: product-catalog
    spec:
      containers:
      - name: product-catalog
        image: fansari9993/test9:product-catalog
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "500m"
          limits:
            cpu: "1"
---
apiVersion: v1
kind: Service
metadata:
  name: product-catalog-service
spec:
  selector:
    app: product-catalog
  ports:
  - protocol: TCP
    port: 8093
    targetPort: 80
    nodePort: 30002
  type: NodePort
```
### Below is my order-processing.yaml file for deployment and service of Order Processing:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-processing
spec:
  replicas: 2
  selector:
    matchLabels:
      app: order-processing
  template:
    metadata:
      labels:
        app: order-processing
    spec:
      containers:
      - name: order-processing
        image: fansari9993/test9:order-processing
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "500m"
          limits:
            cpu: "1"
---
apiVersion: v1
kind: Service
metadata:
  name: order-processing-service
spec:
  selector:
    app: order-processing
  ports:
  - protocol: TCP
    port: 8092
    targetPort: 80
    nodePort: 30003
  type: NodePort
```

### I deployed all the services as can be seen in the below  image:
![alt text](images/Assessment_Day/Image_13)

### I was succesfully able to get my desired webpages as can be seen in the below images.
### This is a frontend webpage:
![alt text](images/Assessment_Day/Image_14)

### This is a Product Catalogue webpage:
![alt text](images/Assessment_Day/Image_15)

### This is a Order Processing webpage:
![alt text](images/Assessment_Day/Image_5)

### After successful I deployment of the images, I pushed the code to the development branch and merged them to the testing and then later to the production branch.

![alt text](images/Assessment_Day/Image_3)

![alt text](images/Assessment_Day/Image_4)

![alt text](images/Assessment_Day/Image_9)
![alt text](images/Assessment_Day/Image_10)
![alt text](images/Assessment_Day/Image_11)
### In the entire project, I didn't use the ConfigMaps or Secrets as no where I needed to enter the username or the password.
