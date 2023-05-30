# Kubernetes
## Create the application with K8s Components

Using **Mongo-Express** and **mongo DB** to create the simple web application set with the help of the K8s components.

### Needs:

1. 2 Deployments / Pods → To create the application and data base.
2. 2 Services → To connect the application through the URL.
3. ConfigMap → To build the deployment.yaml file and contain the database url.
4. Secret → For the credentials It contain credentials.

We create the application with no external service 

This the basic setup of the application.

![flow](https://github.com/Karthikn-n/Kubernetes/assets/102584859/ba3fd7d6-8f89-44a7-a602-f17e4c3cc979)


- It start from the browser send the request to the mongo express external service.
- Then forwarded it to the mongo express pod.
- It convert the request into the internal service.
- It means the mongo db pod.
- And the component take the credential with the request and check and allow to access it.

## Config the Mongo DB and internal Service:

### Step 1:

create the mongo db configuration yaml file.

```yaml
## this file name is mongo.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
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
              name: mongo-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom: 
            secretKeyRef:
              name: mongo-secret
              key: mongo-root-password
```

### step 2:

create the secret configuration file for the security.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque
data:
  mongo-root-username: dXNlcm5hbWU=
  mongo-root-password: cGFzc3dvcmQ=
```

### Step 3:

Note: In a mongo.yaml file we’re referenced to the secret.yaml file for the credentials so It search for the credential inside the cluster.

So apply the secret.yaml file first and then apply the mongo.yaml file

<aside>
🧀 `$ kubectl apply -f mongo-secret.yaml`

</aside>

It create the secret to see that `$ kubectl get secret` .

<aside>
🧀 `$ kubectl apply -f mongo.yaml`

</aside>

To check the it running status  `$ kubectl get pod`

### step 4:

Create Mongo DB **internal Service** config file to connect the application to the database.

```yaml
apiVersion: v1  
kind: Service #This is the service type configuration
metadata:
  name: mongo-service #random name for the file
spec:
  selector:
    app: mongodb #It help to connect to pod through the label
  ports:
    - protocol: TCP 
      port: 27017 #Service port that from the server
      targetPort: 27017 #container port of thdeployment
```

To check the service is configure `$ kubectl get service` use this.

<aside>
🧀 `$ kubectl describe service mongo-service`

</aside>

It shows the IP address the internal mongo-db IP address.

check the mongo-db IP address ,

<aside>
🧀 `$ kubectl get pod -o wide`

</aside>

## Config the mongo-express and external Service:

### Step 1:

Create the express deployment file.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
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
                  name: mongo-secret
                  key: mongo-root-username
            - name: ME_CONFIG_MONGODB_ADMINPASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: mongo-root-password
            - name: ME_CONFIG_MONGODB_SERVER
              valueFrom:
                configMapKeyRef:
                  name: mongo-configmap
                  key: database_url
```

### Step 2:

Crete the configMap file for the application.

```yaml
apiVersion: v1
kind: ConfigMap #type of the configuration
metadata: 
  name: mongo-configmap #name of the file
data:
  database_url: mongo-service #reference to the link
```

### step 3:

Before we did the db configuration same as we will do for this.

- First apply  the configMap file because it is referenced to the application file.

<aside>
🧀 `$ kubectl apply -f monfo-configmap.yaml`

</aside>

Check that is deployed or not using this `$ kubectl get configmap`

- Then deploy the **Mango-express** file

<aside>
🧀 `$ kubectl apply -f mango-express.yaml`

</aside>

to check `$ kubectl get deploment`

### Step 4:

Config the **external service** to connect with the application.

```yaml
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
      nodePort: 30000
```

To check service `$ kubectl get service`.

Here is the **type: LoadBalancer** gives the internal IP address and the External IP address also.

That’s all now use the application using this command.

<aside>
🧀 `$ minikube service mongo-express-service`

</aside>

![Home - Mongo Express - Google Chrome 30-05-2023 00_01_37](https://github.com/Karthikn-n/Kubernetes/assets/102584859/2fc560e7-3987-4d62-93f4-fb093565d7a7)


This is the final output of the application that running on the port number 37239.
