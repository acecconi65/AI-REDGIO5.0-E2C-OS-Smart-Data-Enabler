# Use Case 4: Robotic arm telemetry (large data)

Monitoring of a robotic arm activity

## Use Case reference:
- Large dataset (5000 records)
- IIoT input data in CSV format
- input data read by NiFi pipeline’s processor from a file allocated in a shared network storage area

## Use Case scenario/operation:
In an industrial robotic arm (e.g., an anthropomorphic robot on an assembly line), telemetry monitors not only the environment but also the mechanical and electrical status of individual joints (axes).<br>
Critical variables usually concern the position, current consumption (which indicates stress), and temperature of the motors.<br>
Instead of using SenML or pure JSON format - as done in the other use cases - CSV format will be used here.<br>
Let's not forget the two main reasons why the CSV format is very effective in the field of big data:
- Compactness: Takes up less space for millions of lines.
- Analysis: It can be read directly by software such as Excel, MATLAB or Python libraries such as Pandas.

## IIoT input data shape:
```
timestamp,robot_id,axis_id,pos_deg,current_amp,temp_cel,vibration_mm_s
2025-12-23T16:50:00,ROB-01,1,45.2,1.2,38.5,0.02
2025-12-23T16:50:00,ROB-01,2,-10.5,4.5,42.1,0.05
2025-12-23T16:50:01,ROB-01,1,46.8,1.3,38.6,0.02
2025-12-23T16:50:01,ROB-01,2,-12.1,4.8,42.3,0.08
2025-12-23T16:50:02,ROB-01,1,48.5,2.1,38.8,0.03
2025-12-23T16:50:02,ROB-01,2,-15.0,8.5,43.5,0.15
```

## IIoT input data description and reference values/range used in the demo:
- **timestamp**: time of measurement collection, can be a value between ............. 00:00 and 23:59 on 2025-11-10
- **robot_id**: robot identifier (a fixed value)
- **axis_id**: 1 for x-axis, 2 for y-axis
- **pos_deg**: arm position, expressed in degrees with respect to x-axis or y-axis (indicated by axis_id) - If the value does not correspond to the ‘setpoint’ (theoretical position), there may be a problem with the encoder or a mechanical obstruction.
- **current_amp**: motor current (Ampere) - A sudden increase in current indicates that the motor is working too hard: there may be a lack of lubrication or the motor may be starting to seize up.
- **temp_cel**: motor temperature (°C) - Excessive heat is often the first sign of an impending electrical fault in the motor windings.
- **vibration_mm_s**: vibrations (milliseconds) - High values indicate worn bearings or loose fixing bolts.

<br><br>
The 5000 rows included in the input data file "IIoT-shape-typeC-UC4.csv" have been generated simulating a typical robotic arm work cycle (pick & place or assembly).<br>
Measures refer to the time range 23-12-2025 17:50:00 - 23-12-2025 18:32:00.<br>
In this scenario:
- x-axis (Base) performs wide forward and backward rotations.
- y-axis 2 (Shoulder) undergoes greater stress (high current and vibrations) when extending or lifting a load.
The temperature rises gradually during arm motion and drops during breaks.

***

## Demo configuration:

### Importing the NiFi pipeline:
In NiFi WebUI:
- drag a new Process Group in the canvas<br>
<img width="478" height="126" alt="NiFI-ProcessGroup" src="https://github.com/user-attachments/assets/eb5c908b-a3d1-4901-8ae4-a79b80b49436" /><br>
- click on the right side icon in the Field name:<br> 
<img width="467" height="266" alt="NiFi-Loading" src="https://github.com/user-attachments/assets/973528b6-0a4d-4343-a37f-e62b5d724485" /><br>
- upload file **air5-eda-uc4-pipeline.json** the pipeline will be placed in the canvas:<br>
<img width="346" height="162" alt="NiFi-UC1" src="https://github.com/user-attachments/assets/03aaa9e4-888a-45a2-93d9-91b28ec0c886" /><br>
- double click on the Process Group, the overall detailed pipeline will be shown, with the evidence of all the steps, ready to be activated:<br>
<img width="1664" height="680" alt="NiFi-UC4-pipeline" src="https://github.com/user-attachments/assets/6bf6fddf-b45d-4dc0-80d2-bb34a5252c3e" />
<br>
Step/processor's configuration and properties are reachable right-clicking on the processor itself and selecting "Configure".

### Preparing the input file for the NiFi pipeline:
Thanks to the following directive in the docker-compose.yml file:<br>
<img width="307" height="32" alt="NiFi-UC3-file" src="https://github.com/user-attachments/assets/789d4710-e81a-49c6-b834-edba7cd16c60" /><br>
and having created "source-data" directory for input data (see Deployment Steps - Step 1), let's copy "IIoT-shape-typeC-UC4.csv" file in that directory.<br>
In genaral, you can copy the data related to your specific use case: just make sure it complies to "IIoT input data shape" format and it is included in a file with ".csv" suffix.

### Creating the MinIO bucket for data:
In MinIO WebUI:
- create a bucket named "air5-eda-uc4-bucket"
  
### Creating the InfluxDB bucket for data:
In InfluxDB WebUI:
- create a bucket named "air5-eda-uc4-data"
  
### Importing the Grafana dashboard:
In Grafana WebUI:
- import file **air5-eda-uc4-dashb-1770387944741.json** using the import feature under New menu on the upper right:<br>
<img width="1913" height="235" alt="Grafana-Import" src="https://github.com/user-attachments/assets/982e31d1-b2c8-4d53-aed3-ad263f49ca8e" />

It is important to highlight that the import process includes both the creation of the dashboard analytics templates and the configuration of the connection to InfluxDB data, as it can be quite under:
<img width="1664" height="550" alt="Grafana - Influx" src="https://github.com/user-attachments/assets/d303eb94-db6d-44e4-8ac6-5f90b7618dc8" />

## NiFi pipeline description:
Going back to NiFi WebUI, let's give a detail to all the pipeline steps:
- step 1:  submitting the source JSON dataset to the application
as anticipated in "Preparing the input file for the NiFi pipeline" section above, and as you can check going to the Properties tab of processor's configuration, the source dataset is read from file:<br><br>
<img width="1014" height="604" alt="NiFi-UC3-Step1" src="https://github.com/user-attachments/assets/dc3e61e5-feae-4570-b5dd-e287b190f86d" />

This step leads to two flows: the first one (2a) intends to store original data (for backup or further analysis reasons) into a MinIO storage area, the second one (2b to 8) represents the core application path (from data to analysis)<br><br>
In order to be easily stored into InfluxDB, JSON data are transformed into Line Protocol format - the most suitable data format expected by InfluxDB. As an example, the JSON file in the "IIoT input data shape" section has this Line Protocol representation:<br>
```
environment,deviceId=sensor-R,status=operational temperature=12.4,humidity=6.2,pressure=18.5,battery=3.31 1762772162000000000
```
where the last value represents the epoch (nanoseconds) conversion of timestamp format.<br>
Steps from 2a to 5 implement the transformation "record per record".<br>
- step 2a: storing data into the MinIO bucket
- step 2b: splitting JSON data per single record
- step 3:  extracting single fields from records
- step 4:  normalizing timestamp field
- step 5:  converting JSON data into Line Protocol format (InfluxDB friendly)
- step 6:  merging data to submit to InfluxDB
- step 7:  writing data into InfluxDB
- step 8:  logging InfluxDB transactions

***

## Running the demo:

### NiFi pipeline activation:
In NiFi WebUI:
1. Activate (right-click on the processor, then select "Start": the red square icon will turn into a green arrow icon) all the processors except step 1 "Submitting IoT dataset use case 1"
2. Activate processor step 1 for 2 seconds then deactivate it (just to trigger data start flowing along the pipeline)
3. Check the value of "Tasks/Time" in the processors step 7 and 8: when they are no longer equal to zero, it means that the pipeline is complete

### Sending IIoT data to MinIO:
In MinIO WebUI:
- the input dataset has been sent to the bucket previously created:
<img width="1507" height="347" alt="MinIO-UC3" src="https://github.com/user-attachments/assets/bdc6f6d4-5804-472a-8b5e-bab5cbdcb641" />

### Storing process data in InfluxDB database:
In InfluxDB WebUI:
- going to Data Explorer environment and selecting from 2025-11-10 01:00:00 to 2025-11-10 01:30:00 as the custom time range, it will be possible to query, visualize and navigate through data stored, by applying all the desired filters to data and then visualizing them clicking on SUBMIT button:
<img width="1826" height="831" alt="Influx-UC3" src="https://github.com/user-attachments/assets/e6882315-50ef-4f42-b4df-5750cef22d51" />

### Visualizing analytics with Grafana:
In Grafana WebUI, the dashboard previously imported will provide the following four analytics, all related to the time range 2025-11-10 01:00:00 - 2025-11-10 01:30:00:<br>
<img width="1605" height="724" alt="Grafana-UC4" src="https://github.com/user-attachments/assets/3cf804b7-dff4-41e5-b6d1-94fdf803f1a3" />

Let's examine each analytic in detail.

**1. Overall measures in time range (time series):** this analytic visualizes all the measures included in the given time range.
In the following view, it is highlighted an Out-of-range status for sensor-N caused by a battery value of 3.16 (lower than the threshold of 3.3) occurred at 01:00:34:
<img width="1812" height="827" alt="Grafana-UC3-dashb" src="https://github.com/user-attachments/assets/18b769c3-c763-4191-938c-0fed543c7adb" />

InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc3-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
```
**2. Focus on measures: temperature for sensor-R (time series):** this analytic visualizes the temperature trend for a specific sensor (sensor-R) in the given time range, with evidence of outliers.
In the following view, it is reported the trend on a shorter time range (5 minutes: from 01:00 to 01:00) and it is highlighted a Warning value of 13.7 for temperature occurred at 01:00:13:
<img width="1606" height="829" alt="Grafana-UC3-dashb2" src="https://github.com/user-attachments/assets/f9e13ddb-cdc0-48fc-baed-018389ceae02" />

InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc3-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
|> filter(fn: (r) => r["deviceId"] == "sensor-R")
|> filter(fn: (r) => r["_field"] == "pressure")
|> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
|> yield(name: "pression_phase_R")
```
**3. Focus on measures: humidity (time series):** this analytic visualizes the humidity trend for a specific sensor (sensor-R) in the given time range, with evidence of outliers.
InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc3-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
// Specific filter for sensor R
|> filter(fn: (r) => r["deviceId"] == "sensor-R")
|> filter(fn: (r) => r["_field"] == "humidity")
|> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
|> yield(name: "humidity_phase_R")
```
**4. Measures per operative status in time range (pie chart):** this analytic monitors status changes in the given time range, with evidence of which sensor is reporting statuses other than normal (‘operational’) and how often.
In the following view, it is highlighted the presence of 90 Warning events out of a total of 4000 events:
<img width="1617" height="829" alt="Grafana-Uc4-dashb1" src="https://github.com/user-attachments/assets/ecd75519-b112-49d3-9e6a-a4022b4190d0" />

InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc3-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
|> filter(fn: (r) => r["_field"] == "temperature") 
|> group(columns: ["status"])
|> count()
|> yield(name: "status_count")
