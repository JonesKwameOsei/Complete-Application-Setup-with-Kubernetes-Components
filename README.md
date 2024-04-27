# Complete-Application-Setup-with-Kubernetes-Components
Deploying applications with Kubernetes 
## Overview
In this project, we will deploy two applications, **MongoDB** and **Mongo-Express** with **Kubernetes**. These applications demonstrates well a typical simple setup of **web applications** and their **databases**. 
### Setup
We will create in this project:
- 2 Deployments/Pods
- 2 Services
- 1 ConfigMap
- 1 Secret <p>
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
## Create the Secrete Configuration Files
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
### Create the Secret 
To create the **Secret** credentials, run:
```
kubectl apply -f mongoDB-secret.yaml
```
If the output after running the above is **"secret/mongodb-secret created"**, then the **Secret** has been successfully created. 
We can further confirm this by running:
```
kubectl get secret
```
The resulted output will comfirming the creation of the secret is below:<p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/f588a200-1eb9-408a-8e96-eeea042f8d72)

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
### Create MongoDB Pod 
To create the **MongoDB Pod**, run:
```
kubectl apply -f mongoDB-deployment.yaml
```
The ouput, **"deployment.apps/mongodb-deployment created"** confirms the creation of the pod. 

To view all components, let us run:
```
kubectl get all
```
This command lists all components including the **Cluster-IP**, **Service**, **Pod** and the **replica**. <p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/5b38c2cb-ca7c-4905-923a-c5d360153e90)

##  Create the Service Configuration Files
The following configuration creates the service custom service for the pod. Since the pod and service goes together, we will add the configuration in the same files as the deployment file. 
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
---
# Service Configuration File
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
  # the port that this service should serve on
  - protocol: TCP
    port: 27017
    targetPort: 27017
```
### Create the Service
We will re-run the command below to create the service:
```
kubectl apply -f mongoDB-deployment.yaml
```
By re-running the command, Kubernetes creates only the service and leaves the mongoDB pod unchanged as shown below.<p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/1c3a88b7-b2ae-4aaa-8c92-aca99fae5332)<p>

To confirm the **custom** service creation, we will run:
```
kubectl get service
```
We can observe that the service was created with the following details:
- **Name**: mongodb-service
- **Type**: ClusterIP
- **Cluster-IP**: 10.105.197.33
- **External-IP**: None
- Port(s): 27017/TCP <p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/08b45afe-0ccc-4705-b0c4-0afd6c33361c)<p>

Next, we will investigate if the **service** is attached the **MongoDB Pod**. To do this, we will run:
```
kubectl describe service mongodb-service
```
the output results after running the command:
```
Name:              mongodb-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=mongodb
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.105.197.33
IPs:               10.105.197.33
Port:              <unset>  27017/TCP
TargetPort:        27017/TCP
Endpoints:         10.244.0.12:27017
Session Affinity:  None
Events:            none <p>
```
The results above indicates that the service, **mongodb-service**, is attached to the pod since the **TargetPort** for the service is **27017/TCP** with the **EndPoints** **10.244.0.12:27017**. The endpoint is the **IPAddress** of the pod, 10.244.0.12 and the **port**, 27017, that the application inside the pod is listening to. We can further confirm the IP address of the pod by running:
``` 
kubectl get pod -o wide
```
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/5daab269-38be-416f-aa7f-294bef72e967)<p>
It is evident from the above that, the IP address of the pod is **10.244.0.12**.<p>
Now that the **MongoDB deployment and service** has been created, we can view all components of the pod by running: 
```
kubectl get all | grep mongodb
```
From the output, we can observe that the **Pod**, **service** and **replicaset** were created. However, the **replicaset** was automatically generated by **Kubernetes**. <p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/33a5a0fb-caf5-4662-bcf0-a0639768fc88)<p>

## Create Mongo-Express and ConfigMap
Next, we will create the **Pod** for **Mongo-Express** and its service as well as the External ConfigMap file. The external config files stores the database URL for the MongoDB. <p>

### Create Mongo-Express Configuration Files
The default port for mongo-express is 8081. Mongo-Express will also require a databse to connect to. It is also needs the MongoDB address, Internal Service as well as the credentials for the MongoDB to authenticate. We can set the following **environmental variables** to authnticate the credentials:
- ME_CONFIG_MONGODB_ADMINUSERNAME 
- ME_CONFIG_MONGODB_ADMINPASSWORD          
- ME_CONFIG_MONGODB_SERVER <p>

To set up the configuration file, create a new file and name it **mongoexpress.yaml**. Add the following configuration:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-express
  labels:
    app: mongodb-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb-express
  template:
    metadata:
      labels:
        app: mongodb-express
    spec:
      containers:
        - name: mongodb-express
          image: mongo-express
          ports:
            - containerPort: 8081
          env:
            - name: ME_CONFIG_MONGODB_ADMINUSERNAME
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongo-root-username
            - name: ME_CONFIG_MONGODB_ADMINPASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongo-root-password
            - name: ME_CONFIG_MONGODB_SERVER
              value: 
```

### Create External ConfigMap Files
Next, we will create the configuration map file to store the **MongoDB** database URL. The credentials will be referenced in the **MongoExpress** configuration file. In a new file, **mongo.configmap.yaml, add:
```
# Defining the configmap file
apiVersion: v1
kind: ConfigMap
metadata: 
  name: mongodb-configmap
data: 
  # Referencing the MongoDB service namespace 
  database_url: mongodb-service
```

### Referencing Credentials in ConfigMap to MongoExpress 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom: 
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom: 
            configMapKeyRef: # referenced the configmap 
              name: mongodb-configmap
              key: database_url
```
### Create MongoExpress Pod and ConfigMap
We will create the **configmap** before the **mongoexpress** pod. Run:
```
kubectl apply -f mongo-configmap.yaml
kubectl apply -f mongoExpress.yaml
```
The image below indicates that our ConfigMap and MongoExpress were created successfully.<p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/653e4394-03d2-4b4e-b40e-7238cea1b2a1)<p>
 We will run: 
 ```
kubectl get pods
```
This command enbles us to see the MongoExpress created and running.<p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/d02a4c58-7763-457f-b2b1-a6ed829ad224)<p>

We have **Mongo-Express** running, let us check its components by running:
```
kubectl get all | grep mongo-express
```
The outcome indicates that the **pod**, **deployment** and the **replicaset** have been created. <p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/6c09dbd0-0754-4b10-bfe1-36d4aa600fe6)<p>

Now that we have checked the components of **Mongo-Express**, we will further investigate if it is connected to the **MongoDB** database and service. We can achieve this by running: 
```
kubectl logs mongo-express-859f75dd4f-pjw77
```
It can be observed that **Mongo-Express** is running on version **1.0.2** and the **Mongo Express server** is listening at **port 8081**. <p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/c1bedcc5-8754-4bd2-878b-b1ce2c278456)<p>

### Create Mongo Express External Service
Mongo Express is a **web-based MongoDB admin interface**, which allows users to manage **MongoDB databases** and collections through a graphical user interface (GUI) accessible through a web browser.<p>

By running **Mongo Express** as an external service, it becomes accessible from any device that has a web browser, making it easier to manage the **MongoDB databases** from multiple locations.<p>

We will create the **Mongo-express** service configuration in the same file as the pod (Mongo-Express):
```
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service 
spec:
  selector:
    app: mongo-express
  type: LoadBalancer
  ports: 
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 32000
```
The **type: LoadBalancer** directs external traffic to the pod. To create the service, we will run:
```
kubectl apply -f mongoExpress.yaml
```
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/ed894668-b216-41cc-9b01-671d4619ba25)<p>

Let's compare of **TYPE** of the **mongodb-service** to that of the **mongo-express-service** by running:
```
kubectl get service | grep mongo
```
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/c009efe8-39b4-47e8-ba88-845018ecc9b2)<p>
From the above image, whereas **mongodb-service** has **ClusterIP** as its **TYPE**, **mongo-express-service** has its **TYPE** as a **LoadBalancer**. The main differences between clusterIP and loadbalancer service types is that **clusterIP** services are only accessible within the **Kubernetes cluster**, while **loadbalancer** services are accessible from the public internet. <p>

**N/B**: ClusterIP (Internal Service) is **Default** and we do not need to declare it in the configuration. 

From the image above, we can also observe that the external-ip address for the **mongo-express-service is <pending>. To get the External-IP address, we have to run:
```
minikube service mongo-express-service
```
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/c48d2a3a-240b-473e-bc1e-eeb1d9fe1308)<p>
The browser opens requesting the MongoDB credentials. <p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/6a9788f4-1afc-4394-a991-645a3de6657e)<p>
MongoExpress is now accessible on the web.<p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/af725941-ac17-4673-88ad-451aa2a3aa44)<p>

### Create A Database in MongoExpress GUI
To test that the **Mongo-Express** is communicating to the **MongoDB** database, we will create a new database in the GUI called **JonesDB**. In the **database name** field, enter **JonesDB** and click on **+ Create Database** next to it to create the new database. <p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/f744c27c-ca38-42e7-8924-c27ce5bd099d)<p>
![image](https://github.com/JonesKwameOsei/Complete-Application-Setup-with-Kubernetes-Components/assets/81886509/81cdb884-de85-4577-a82d-8f8117312b5e)

## Conclusion
In this project, we utilised **Kubernetes** to orcherstrate containers. We setup and deployed MongoDB database and Mongo-Express. We created MongoDB database with its service, ensured its security with secrets and accessed the database externally on the web through Mongo-Express. 

























