# AI-REDGIO5.0-E2C-OS-Smart-Data-Enabler

Smart Data Enabler is a basic stack consisting in an AI Edge-to-Cloud (E2C) minimum viable application using AI REDGIO 5.0 recommendend Open Source (OS) tools. 

It is a technological demo showing how a very simple architecture based on open source tools recommended in the project allows SMEs to obtain useful analytical insights from their process data even in conditions of limited maturity and complexity, with reduced technological skills and practically zero costs/investments.

This guide explains how to deploy the application using Docker Compose. 
The application is packaged as a prebuilt Docker image, and all necessary configurations are provided in the .env file.

## Architecture




## Prerequisites
Before you begin, ensure the following are installed on your system:

1. Docker:

Install Docker by following the official guide: [Install Docker](https://docs.docker.com/get-started/get-docker/).

2. Docker Compose:

Install Docker Compose by following the official guide: [Install Docker Compose](https://docs.docker.com/compose/install/).

***

## Deployment Steps

### Step 1: Prepare the Deployment Directory

Create a directory for the deployment files:
```
mkdir _your-app-deployment_
cd _your-app-deployment_
```
Copy the following files into the directory:
- docker-compose.yml (provided in this repository)
- env.example (provided in this repository)

Create a directory for input data:
```
mkdir source-data
```

### Step 2: Environment Variables

You need to fill all the environmental variables that missing in the env.example file and create a .env file.

The application uses environment variables which are defined in the .env file.

Ensure the .env file is in the same directory as the docker-compose.yml file.

For the variables that have a comment to replace them please use something secure, if not the application will deployed properly but with default values. 
When you add all the variables you need copy env.example and create the .env file using the following command Linux/MacOs
```
cp env.example .env
```
### Step 3: Run the Application
Start the application using Docker Compose:
```
docker-compose up -d
```
### Step 4: Access the application

- NiFi:
Open a browser, navigate to [https://localhost:8443/nifi](https://localhost:8443/nifi) then access with credentials provided in the .env file.
- MinIO:
Open a browser, navigate to [https://localhost:9001](https://localhost:9001) then access with credentials provided in the .env file.
- InfluxDB:
Open a browser, navigate to [https://localhost:8086](https://localhost:8086) then access with credentials provided in the .env file.
- Grafana:
Open a browser, navigate to [https://localhost:3000](https://localhost:3000) then access with credentials provided in the .env file.

### Stopping the Application
To stop the application, run:
```
docker-compose down
```
