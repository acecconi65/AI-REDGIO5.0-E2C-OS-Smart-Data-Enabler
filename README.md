# AI-REDGIO5.0-E2C-OS-Smart-Data-Enabler

Smart Data Enabler is a basic stack consisting in an AI Edge-to-Cloud (E2C) minimum viable application using AI REDGIO 5.0 recommendend Open-Source (OS) tools. 

It is a technological demo showing how a very simple architecture based on open source tools recommended in the project allows SMEs to obtain useful analytical insights from their process data even in conditions of limited maturity and complexity, with reduced technological skills and practically zero costs/investments.

## Scenario

Smart Data Enabler (SDE) application goal is to demonstrate how a SMEs can easily run the following steps in order to conduct an initial exploration and analysis of their (edge) production data. 

<img width="499" height="347" alt="SDA-UC" src="https://github.com/user-attachments/assets/4d30920b-940e-47c5-acde-972eaa215a2f" />

Production data can be a JSON file like this:
```
{
  "operation": "shift",
  "spec": {
    "deviceId": “sensor-001",
    "timestamp": "2025-12-10 10:56:02",
    "readings": {
      "temperature": "12.4",
      "humidity": "4.2",
      "pressure": "18.5",
      "battery": "-1.12"
    },
    "status": "OK"
  }
}
```

## Architecture

The application stack is composed by the following open-source technologies, whose (integrated) usage is recommended by AIREDGIO 5.0 project:
- [Apache NiFi](https://nifi.apache.org/): easy to use, powerful, and reliable open-source system to process and distribute data, particularly suitable for integrating IoT data sources
- [MinIO](https://www.min.io/): high-performance, software-defined Object Storage server, a sort of an open-source, private version of Amazon S3
- [InfluxDB](https://www.influxdata.com/): specialized open-source database designed to handle data that is indexed by time, fitting best when capturing streams of measurements coming from a sensors
- [Grafana](https://grafana.com/): open-source visualization and analytics platform, allowing you to query, visualize, alert on, and understand your metrics no matter where they are stored

and this is how the scenario turns into architecture, leveraging the previous technologies:

<img width="728" height="437" alt="SDA-AR" src="https://github.com/user-attachments/assets/6d2f58bd-c47c-4373-b79f-4052da3a0750" />

Nevertheless, the architecture is ready for various types of improvements thanks to AI.

## Use Cases

Smart Data Enabler application consists of a base stack and four use cases on top:
|Nr|                                   Title|                                    Folder|                                                                                                              Type|        NiFi pipeline| MinIO bucket| InfluxDB bucket| Grafana dashboard|                                                     
|-:|---------------------------------------:|-----------------------------------------:|-----------------------------------------------------------------------------------------------------------------:|--------------------:|-------------:|----------------------------:|---------------------------:|
| 1|Electrical panel monitoring (small data)|uc1-electrical_panel_monitoring-small_data|Small dataset (100 records), IIoT shape type A, JSON format, data directly injected into NiFi pipeline’s processor|air5-eda-uc1-pipeline|air5-eda-uc1-bucket|air5-eda-uc1-data|air5-eda-uc1-dashb|
| 2|    Data Center environmental monitoring|uc2-dc_environmental_monitoring-small_data|Small dataset (100 records), IIoT shape type B, JSON format, data directly injected into NiFi pipeline’s processor|air5-eda-uc2-pipeline|air5-eda-uc2-bucket|air5-eda-uc2-data|air5-eda-uc2-dashb|
| 3|Electrical panel monitoring (large data)|   uc3-electrical_panel_monitoring-large_data|Large dataset (4000 records), IIoT shape type A, JSON format, data read from file on disk|air5-eda-uc3-pipeline|  air5-eda-uc3-bucket|   air5-eda-uc3-data|air5-eda-uc3-dashb|
| 4|                         Robotic arm telemetry|   uc4-robotic_arm_telemetry-large_data|Large dataset (10000 records), IIoT shape type C, CSV format, data read from file on disk|air5-eda-uc4-pipeline|  air5-eda-uc4-bucket|air5-eda-uc4-data|air5-eda-uc4-dashb|

The following content is a guide explaining how to deploy the application using Docker Compose. 
The application is packaged as a prebuilt Docker image, and all necessary configurations are provided in the .env file.
Then you can follow the Use Cases folders provided in order to easily prepare and run a specific use case. 

(DIRE CHE LO SCOPO DI QUESTA APPLICAZIONE E' ANCHE QUELLO DI INVITARE A CREARE IL PROPRIO USE CASE) 

***

## Prerequisites
Before you begin, ensure the following are installed on your system:

1. Docker:

Install Docker by following the official guide: [Install Docker](https://docs.docker.com/get-started/get-docker/).

2. Docker Compose:

Install Docker Compose by following the official guide: [Install Docker Compose](https://docs.docker.com/compose/install/).

***

## Deployment Steps

### Step 1: Prepare the Deployment Directory

Create a directory for the deployment files (replace _your-app-deployment_ with a name of your choice):
```
mkdir your-app-deployment
cd your-app-deployment
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
Open a browser, navigate to [http://localhost:9001](http://localhost:9001) then access with credentials provided in the .env file.
- InfluxDB:
Open a browser, navigate to [http://localhost:8086](http://localhost:8086) then access with credentials provided in the .env file.
- Grafana:
Open a browser, navigate to [http://localhost:3000](http://localhost:3000) then access with credentials provided in the .env file.

### Stopping the Application
To stop the application, run:
```
docker-compose down
```
