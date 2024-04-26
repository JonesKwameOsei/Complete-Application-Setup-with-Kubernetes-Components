# Complete-Application-Setup-with-Kubernetes-Components
Deploying applications with Kubernetes 
## Overview
In this project, we will deploy two applications, **MongoDB** and **Mongo-Express** with **Kubernetes**. These applications demonstrates well a typical simple setup of **web applications** and their **databases**. 
### Setup
We will create in this project:
- 2 Deployments/Pods
- 2 Services
- 1 ConfigMap
- 1 Secret
First, we will create a **MongoDB** Pod. We will need a service to communicate with the pod. This service will be **internal**, which meansno external requests are allowed to the pod. However, only components within or inside the **cluster** can communicate to it. <p>
Next, we will create a **Mongo-Express** pod. Here, we will require **a database URL** of **mongoDB** that the **mongo-express** can connect to it. Alos, we will need the credentials (username and password) of the database (mongoDB) for authentication. We will utilise **deployment configuration file** through **environmental viariables** to pass this **information (credentials)** to the **MongoExpress**. For this to be achieved, we will create a **config map** that contains the **database URL**and a secret that contains the credentials. Then both will be referenced in the **deployment configuration file**. Finally, an **external service (ingress)** will be created to be enable the **MongoExpress** to be externally accessible and external requests to reach and communicate with the pod.
## Create MongoDB Deployment Configuration File
1. In a text editor or IDE (VSCode), create a file called mongoDB-deployment.yaml
```
touch mongoDB-deployment.yaml
``` 
3. In the file created, add the following configuration:
```
"apiVersion": apps/v1
kind:  Deployment
metadata:
  name: mongoDB-deployment
  labels:
    app: mongoDB
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongoDB
  template:
    metadata:
      labels:
        app: mongoDB
    spec:
      containers:
      - name: mongoDB
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value:
        -  name: MONGO_INITDB_ROOT_PASSWORD
          value:
```
