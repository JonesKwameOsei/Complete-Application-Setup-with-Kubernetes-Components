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
# Defining MongoDB Pod
apiVersion: apps/v1
kind:  Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value:
        -  name: MONGO_INITDB_ROOT_PASSWORD
          value:
```
## Create the Secrete Files
To **secure** the credentials, It is advisable to reference the credentials of the MongoDB database in a **secret file** for the following reasons:
1. **Security**: Storing **sensitive** information, such as database credentials, directly in the **Kubernetes manifests** or **application code** can be a security risk. If the Kubernetes manifests or application code is accidentally leaked or accessed by unauthorized parties, it could expose the database credentials, leading to potential data breaches or unauthorized access to the database.

2. **Separation of Concerns**: By separating the database credentials from the application code or Kubernetes manifests, it means the principle of separation of concerns is applied. This ensures that the application and the infrastructure-related configurations are decoupled, making it easier to manage and maintain the application.

3. **Flexibility**: **Referencing** the database credentials in **secret files** makes it easy to update or rotate the credentials without having to modify the Kubernetes manifests or application code. This can be especially useful when it is needed to change the credentials due to security reasons or compliance requirements.

4. **Standardization**: Storing sensitive information, such as database credentials, in Kubernetes secrets is a widely recommended and standard practice in the Kubernetes ecosystem. This approach aligns with the best practices and guidelines for managing secrets in a Kubernetes environment.

5. **Access Control**: Kubernetes provides fine-grained access control mechanisms, allowing DevOps engineers to control which Kubernetes objects and components can access the secret files containing the database credentials. This helps enhance the overall security of your application and data.<p>

Create a new file named **mongoDB-secret.yaml** and configure it as follows:
```
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  mongo-root-username: 
  mongo-root-password: 
```
### Generate Secret Credentials 
Storing the credentials data in a **Secret** component does not automatically make it scure. There should be another layer of security to be put in place. This is because, there are some built-in mechanism (like encription) for basic security, which are not enabled by default. Due to this reason, it is not a good practice to write the credentials in plain texts. Using base64 encoded is a good way to **obsfucate** the secret credentials. In the VScode terminal, run:
```
echo -n 'username' | base64
echo -n 'password' | base64
```
Copy and paste the outputs in the right allocations. 
**N/B**: the 'username' and 'password'can be any secret values you choose. <p>
The mongoDB-secret.yaml should now look like this:
```
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  mongo-root-username: am9uZXNvc2Vpa3dhbWU=
  mongo-root-password: MTRBcG9zdGxlc0A=
```
### Referencing the Secret Values in the Deployment Configuration files
Now we will set the environmental values for the deployment configuration file by referncing the values of the **username** and **password** in **Secret** config files. The file should look like his after referencing:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
```




