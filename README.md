# Node.js App Deployment on Minikube using GitHub Actions

This project demonstrates how to deploy a simple Node.js application on a Minikube Kubernetes cluster using **GitHub Actions** for CI/CD automation. The workflow builds the Docker image, loads it into Minikube, and deploys the app automatically.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Folder Structure](#folder-structure)
3. [Step 1: Setup GitHub Actions](#step-1-setup-github-actions)
4. [Step 2: Create Node.js Application](#step-2-create-nodejs-application)
5. [Step 3: Dockerize the Application](#step-3-dockerize-the-application)
6. [Step 4: Kubernetes Deployment and Service](#step-4-kubernetes-deployment-and-service)
7. [Step 5: Running the Workflow](#step-5-running-the-workflow)

---

## Project Overview
This project automates deployment of a Node.js application to a local Minikube Kubernetes cluster using GitHub Actions.  
Key steps include:
- Building a Docker image of the Node.js app
- Loading the image into Minikube
- Deploying the app with Kubernetes Deployment and Service YAMLs
- Verifying pod status and logs
- Accessing the app via Minikube service

---

## Folder Structure

├── .github/workflows/deploy-minikube.yml # GitHub Actions workflow
├── Dockerfile # Docker configuration for Node.js app
├── k8-node-app.yaml # Kubernetes Deployment + Service
├── package.json # Node.js dependencies and scripts
└── src/server.js # Node.js application code


## Step 1: Setup GitHub Actions

The workflow file `.github/workflows/deploy-minikube.yml` automates the deployment on push to the repository.

## What happens in this workflow:
1. Checkout the repository code.
2. Log in to Docker Hub (required if using private images).
3. Start Minikube on the runner machine.
4. Configure Docker to use Minikube's Docker daemon and load the app image.
5. Deploy the Node.js app using `kubectl apply`.
6. Wait for pods to be ready and check logs for debugging.
7. Test the service URL to ensure the app is accessible.

This allows **automatic deployments** to Minikube whenever code is pushed.

---

## Step 2: Create Node.js Application

The Node.js app is a simple Express server:

```javascript
const express = require('express');
const PORT = 3000;
const HOST = '0.0.0.0';
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(PORT, HOST, () => {
  console.log(`Running on http://${HOST}:${PORT}`);
});
```
This app listens on port 3000 and responds with "Hello World" to any HTTP GET request at /. This is the app that will be deployed in Kubernetes.

## Step 3: Dockerize the Application

Dockerfile:

FROM node:14
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD [ "node", "src/server.js" ]

Sets the base image to Node 14.

Copies package files and installs dependencies.

Copies the app code.

Exposes port 3000.

Starts the app with node src/server.js.

Docker allows us to package the Node.js app with all dependencies, making it portable and ready for Kubernetes deployment.

## Step 4: Kubernetes Deployment and Service

Kubernetes YAML (k8-node-app.yaml):

Deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - name: nodejs-app
        image: "vtejaswinig/nodejs-app:latest"
        ports:
          - containerPort: 3000
```
Service:
```
apiVersion: v1
kind: Service
metadata:
  name: nodejs-app
spec:
  selector:
    app: nodejs-app
  type: NodePort
  ports:
    - port: 80
      targetPort: 3000
```

Deployment creates pods running the Node.js app.

Service exposes the pods via a NodePort so the app is accessible externally through Minikube.

This separation ensures scalability and proper network routing in Kubernetes.

## Step 5: Running the Workflow

Once the workflow is pushed to GitHub:

GitHub Actions triggers the workflow.

The Docker image is built and loaded into Minikube.

Kubernetes applies the deployment and service.

GitHub Actions waits for the pod to become ready.

You can access the Node.js app using:

```
minikube service nodejs-app --url
```
This provides the live URL to access the app running in your local Minikube cluster.
