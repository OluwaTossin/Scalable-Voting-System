# Scalable Voting System: Deploying a Multi-Tier Application on Kubernetes

![image](https://github.com/OluwaTossin/Scalable-Voting-System/assets/121174963/1a951e21-1bd6-4bb0-ae67-c6c777fbdd92)

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
- Inside the "voting app" directory, I crafted a deployment configuration file named voting-app-deploy.yaml.
- The contents of this file are as follows:

`apiVersion: apps/v1
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
            - containerPort: 80`

