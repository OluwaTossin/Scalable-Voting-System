# Scalable Voting System: Deploying a Multi-Tier Application on Kubernetes

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/c17eb6cf-4916-48f3-b38d-93935001ce82)


After concluding the Kubernetes class on @kodedkoud, I've successfully deployed the Voting App on Kubernetes using Deployments. This class project allowed me to apply what I learned, integrating multiple tech stacks like Python, Redis, .NET, PostgreSQL, and Node.js. Deploying it on Kubernetes emphasized the importance of container orchestration in real-world scenarios.

## Components and their interactions

Here's a detailed breakdown of its components and how they interconnect:
1.	voting-app (Python):
- Functionality: Allows users to cast their votes between two options.
- Interaction: It communicates directly with the Redis database, storing the voting data for processing.
2.	redis (Redis):
- Functionality: An in-memory data structure store that serves as a temporary data store and message broker for our system.
- Interaction: It holds the voting data temporarily, which the worker service then picks up for further processing.
3.	worker (.NET):
- Functionality: This is our background processing unit. It's responsible for collecting, analyzing, and then saving the processed data.
- Interaction: The worker retrieves the voting data from Redis, processes the votes, and subsequently saves the results into the PostgreSQL database.
4.	db (PostgreSQL):
- Functionality: Serves as the persistent storage system for our application, ensuring long-term data safety and integrity.
- Interaction: The result-app directly queries this database to retrieve and display the final voting results to the users.
5.	result-app (Node.js):
- Functionality: It's our frontend component, designed to present users with the results of their votes in a clear, intuitive format.
- Interaction: It communicates with the PostgreSQL database to fetch the voting results and present them to the users.
Each of these components is containerized and managed by Kubernetes, ensuring scalability, high availability, and seamless interaction.

## Steps:

## Step 1: Setting up the Project Directory
- Initiated the project by creating a directory named "voting app".
- Within this directory, I generated individual pod definition files for each component of the application.

## Step 2: Deployment Configuration for the Voting App
- Inside the "voting app" directory, I crafted a deployment configuration file named voting-app-deploy.yaml and a voting-app-service.yaml.
- The contents of this file are as follows:

voting-app-deploy.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-app-deploy
  labels:
    name: voting-app-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: voting-app-pod
      app: demo-voting-app
    
  template:
    metadata:
      name: voting-app-pod
      labels:
        name: voting-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: voting-app
          image: kodekloud/examplevotingapp_vote:v1
          ports:
            - containerPort: 80
```

voting-app-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: voting-service
  labels:
    name: voting-service
    app: demo-voting-app
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
  selector:
    name: voting-app-pod
    app: demo-voting-app

```

## Step 3:
Next is to create a redis deployment and its service. Create a definition file with the below:

redis-deploy.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deploy
  labels:
    name: redis-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: redis-pod
      app: demo-voting-app
    
  template:
    metadata:
      name: redis-pod
      labels:
        name: redis-pod
        app: demo-voting-app
    spec:
      containers:
        - name: redis
          image: redis
          ports:
            - containerPort: 6379
```

redis-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
    name: redis-service
    app: demo-voting-app
spec:
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    name: redis-pod
    app: demo-voting-app
```

## Step 4: create a postgress.deployment.yaml
- For the database layer of our application, I set up a PostgreSQL deployment.
- The deployment configuration was established in a file named postgres-deployment.yaml and service in a postgres-service.yaml configuration file

postgres-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deploy
  labels:
    name: postgres-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: postgres-pod
      app: demo-voting-app
    
  template:
    metadata:
      name: postgres-pod
      labels:
        name: postgres-pod
        app: demo-voting-app
    spec:
      containers:
        - name: postgres
          image: postgres
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_USER
              value: "postgres"
            - name: POSTGRES_PASSWORD
              value: "postgres"
            - name: POSTGRES_HOST_AUTH_METHOD
              value: trust

```
postgres-service.yaml 

```
apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    name: postgres-service
    app: demo-voting-app
spec:
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    name: postgres-pod
    app: demo-voting-app

```
## Step 5: Next is to create the worker-app-deployment.yaml
- To process and handle backend operations, a Worker App deployment was created.
- This was defined in the worker-app-deployment.yaml file, structured as:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker-app-deploy
  labels:
    name: worker-app-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: worker-app-pod
      app: demo-voting-app
    
  template:
    metadata:
      name: worker-app-pod
      labels:
        name: worker-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: worker-app
          image: kodekloud/examplevotingapp_worker:v1

```
## Step 6: Deployment Configuration and service for Result Application
- Lastly, I drafted a deployment configuration for the Result Application, which displays voting outcomes to users.
- This configuration was saved in the result-app-deploy.yaml file and result-app-service.yaml.

result-app-deploy.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result-app-deploy
  labels:
    name: result-app-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: result-app-pod
      app: demo-voting-app
    
  template:
    metadata:
      name: result-app-pod
      labels:
        name: result-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: result-app
          image: kodekloud/examplevotingapp_result:v1
          ports:
            - containerPort: 80

```
result-app-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: result-service
  labels:
    name: result-service
    app: demo-voting-app
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
  selector:
    name: result-app-pod
    app: demo-voting-app
```
With all deployment definition files ready, it's time to switch to the terminal. Here, I'll initiate the deployment processes and configure their respective services for seamless interactions.

## Terminal Steps for Deployment

## Step 1:
Begin by ensuring the cluster is free from any non-default pods and services.

`kubectl get pods`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/fe8da23e-48a2-498a-a988-07cfccb4bd80)

From my observations, there are no extra pods and services active other than those that come default with Kubernetes.

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/ae224738-9d62-4bec-a3f6-b2eab3369f56)

## Step 2: Deployment of the Voting App
Initiate the deployment for the voting app using:

`kubectl create -f voting-app-deployment.yaml`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/16cc008e-e83c-4daa-be84-bb87b177facf)

Once the deployment is completed, proceed with creating the associated service:

`kubectl create -f voting-app-service.yaml`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/35c09ad0-e89e-4800-8060-be0b620bf5a7)

It's crucial to verify that our deployments are functioning as intended. Check their status with:

`Kubectl get deployment`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/0759b172-8c91-4708-acb2-7d23e3c85e00)

The output should indicate that the voting app is up and operational.

## Step 3: Deployment of Redis
Now, let's establish the Redis deployment using:

`kubectl create -f redis-deploy.yaml`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/91c49cb6-bc77-44b5-b833-5516e28ac3d3)

Once deployed, it's time to create the corresponding service for Redis:

`kubectl create -f redis-service.yaml`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/a129ebe2-0a5b-4b29-82f3-743b440d05ab)

## Step 4: Deployment of PostgreSQL

Deploy PostgreSQL using the following command:

`kubectl create -f postgres-deploy.yaml`

Once the deployment has completed, initiate the associated service:

`kubectl create -f postgres-service.yaml`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/1705c1da-8964-4098-a727-dd796c427a2c)

Ensure all deployments are active and healthy:

`Kubectl get deployments`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/af54673e-a60f-4879-8f67-176b8f3b8f2e)

Also, validate the services associated with the deployments:

`kubectl get service`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/cdd87b74-82ef-4b41-820c-03ef78525e68)

## Step 5: Worker and Result App Deployment

Next, deploy the result application:

`kubectl create -f result-app-deploy.yaml`

And the service:

`kubectl create -f result-app-service.yaml`

Deploy the worker application:

`kubectl create -f worker-app-deploy.yaml`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/707c82ec-843b-4947-b84c-d237e1debb41)

confirm all services are setup

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/b92a0133-6961-4291-9a82-6738ae045f9a)

## Step 6: Final Confirmation

To be certain, do a final check of both the deployments and services to confirm everything is in order:

`Kubectl get deployments`

`kubectl get service`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/60fdd28b-a896-45d4-b708-6173a9ee6798)

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/4e62b8dd-9199-463e-872a-9cb3b36f3277)

## Step 7: Accessing the Services

To access the voting frontend service locally, use the following command:

`kubectl port-forward service/voting-service 8080:80`

For accessing the result backend service locally, utilize:

`kubectl port-forward service/result-service 8081:80`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/28b8ecbc-3074-431e-8767-85865696a69a)

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/52aef43a-c2f1-4f32-99ea-d92297cacb6a)

## Step 8: Scaling the Application

If there's a need to accommodate more traffic, consider scaling up the voting app deployment. For instance, to run three replicas, execute:

`kubectl scale deployment voting-app-deploy --replicas=3`

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/880491c1-0511-4ba1-ba46-4252d837a6a2)

To validate that the scaling took effect, check the number of pods:

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/e15a99bb-be5f-4e6d-94d4-0f004d163582)

This should confirm that there are three instances of the voting app.

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/a30a5050-d95d-43ad-b3b4-788a58f35dca)

With this setup, every time the voting page is refreshed, it is highly likely that a different pod will serve the request, showcasing the load distribution across the replicas.
