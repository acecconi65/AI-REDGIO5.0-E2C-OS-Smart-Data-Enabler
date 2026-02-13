# Use Case 1: Electrical panel monitoring (small data)

Monitoring of a critical electrical panel (three-phase electrical system)

## Use Case reference:
- Small dataset (100 records)
- IIoT input data in JSON format
- input data directly injected into a NiFi pipeline’s processor 

## Use Case scenario/operation:
Four IIoT sensors – one for each phase R, S, T and Neutral - positioned inside a critical high-voltage substation wide electrical panel, a cooled environment for which it is fundamental to check data in order to highlight anomalies.<br>
In highly critical industrial or infrastructure scenarios (hospitals, data centres, power stations), the use of a single sensor is often considered a single point of failure.<br>
Installing four sensors on a single electrical panel allows for the implementation of redundancy and differentiated monitoring logic.<br>
The objective is to detect whether one of the terminals is loosening (causing an increase in resistance and therefore heat) before an electric arc is triggered: if the temperature of phase R is significantly higher than the others, the system generates a predictive maintenance alarm.

## IIoT input data shape:
```
{
  "operation": "EP-G02-monitoring",
  "spec": {
    "deviceId": “sensor-R",
    "timestamp": "2025-11-10 10:56:02",
    "telemetry": {
      "temperature": "12.4",
      "humidity": "6.2",
      "pressure": "18.5",
      "battery": "3.31"
    },
    "status": "operational"
  }
}
```

## IIoT input data description and reference values/range used in the demo:
- **operation**: representing the process operation, always equal to "EP-G02-monitoring"
- **device ID**: one of the four EP sensors’ serial code, can be one of “sensor-R" o “sensor-S" o “sensor-T" o “sensor-N"
- **timestamp**: time of measurement collection, can be a value between 00:00 and 23:59 on 2025-11-10
- **temperature**: the optimal operational range is 10.0-13.0 [°C], indicating a cooled environment
- **humidity**: the optimal operational range is 5.0-7.4 mbar for low (10.0 °C) temperatures, that is to be maintained to prevent electric arcs or partial discharges due to condensation and prevent both condensation and electrostatic discharge - at 10 °C, the air is ‘saturated’ (i.e. it reaches 100% humidity and begins to form condensation) when the water vapour pressure reaches approximately 12.3 mbar.
- **pressure**: the optimal operational range is 10.0-20.0 mbar – in highly critical electrical cabinets, the interior of the cabinet is maintained at a pressure slightly higher than the external pressure using nitrogen or filtered dry air - if the pressure drops, it means that the seals on the panel are worn or that a door has been left open - maintaining positive pressure prevents conductive dust, corrosive vapours or external moisture from entering, drastically reducing the risk of electric arcs.
- **battery**: the optimal operational range is 3.3-3.6 V for low (10.0 °C) temperatures – values lower than 3.0 V have to be considered as alarm, requiring immediate technical intervention
- **(device) status**:  describing normal or alert/critical situations related to the values detected (voltage, current, terminal temperature):
  - “operational”: normal 
  - “warning”: the value is outside the optimal range, but the system is still safe
  - “critical”: an imminent danger threshold has been exceeded
  - “out of range”: the sensor is detecting a value that exceeds its physical measurement capabilities (often indicating a transducer failure)

***

## Demo configuration:

### Importing the NiFi pipeline:
In NiFi WebUI:
- drag a new Process Group in the canvas<br>
<img width="478" height="126" alt="NiFI-ProcessGroup" src="https://github.com/user-attachments/assets/eb5c908b-a3d1-4901-8ae4-a79b80b49436" /><br>
- click on the right side icon in the Field name:<br> 
<img width="467" height="266" alt="NiFi-Loading" src="https://github.com/user-attachments/assets/973528b6-0a4d-4343-a37f-e62b5d724485" /><br>
- upload file **air5-eda-uc1-pipeline.json** the pipeline will be placed in the canvas:<br>
<img width="346" height="162" alt="NiFi-UC1" src="https://github.com/user-attachments/assets/03aaa9e4-888a-45a2-93d9-91b28ec0c886" /><br>
- double click on the Process Group, the overall detailed pipeline will be shown, with the evidence of all the steps, ready to be activated:<br>
<img width="1583" height="628" alt="NiFi-UC1-pipeline" src="https://github.com/user-attachments/assets/d15ca0de-8e4f-40b4-9ad2-25cf10c37846" />
<br>
Step/processor's configuration and properties are reachable right-clicking on the processor itself and selecting "Configure".

### Creating the MinIO bucket for data:
In MinIO WebUI:
- create a bucket named "air5-eda-uc1-bucket"
  
### Creating the InfluxDB bucket for data:
In InfluxDB WebUI:
- create a bucket named "air5-eda-uc1-data"
  
### Importing the Grafana dashboard:
In Grafana WebUI:
- import file **air5-eda-uc1-dashb-1770387903460.json** using the import feature under New menu on the upper right:<br>
<img width="1913" height="235" alt="Grafana-Import" src="https://github.com/user-attachments/assets/982e31d1-b2c8-4d53-aed3-ad263f49ca8e" />

It is important to highlight that the import process includes both the creation of the dashboard analytics templates and the configuration of the connection to InfluxDB data, as it can be quite under:
<img width="1664" height="550" alt="Grafana - Influx" src="https://github.com/user-attachments/assets/d303eb94-db6d-44e4-8ac6-5f90b7618dc8" />

## NiFi pipeline description:
Going back to NiFi WebUI, let's give a detail to all the pipeline steps:
- step 1:  submitting the source JSON dataset to the application
as anticipated in "Use Case reference" section above, and as you can check going to the Properties tab of processor's configuration and right-clicking on "Custom text" field, the source dataset is directly inserted internally to the pipeline:<br><br>
<img width="1270" height="612" alt="NiFi-CustomText" src="https://github.com/user-attachments/assets/753c2d64-c9f2-4081-b66a-6f8b6335d88c" /><br><br>
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
3. Check the value of "Tasks/Time" in the processors step 7 and 8: when they are no longer equal to zero, it means that the pipeline has finished

### Sending IIoT data to MinIO:
In MinIO WebUI:
- the input dataset has been sent to the bucket previously created:
<img width="1620" height="402" alt="MinIO-file" src="https://github.com/user-attachments/assets/b7107ce9-13a9-45b3-bd0f-9756845ecbce" />

### Storing process data in InfluxDB database:
In InfluxDB WebUI:
- going to Data Explorer environment and selecting from 2025-11-10 00:00:00 to 2025-11-10 23:59:00 as the custom time range, it will be possible to query, visualize and navigate through data stored, by applying all the desired filters to data and then visualizing them clicking on SUBMIT button:
<img width="1893" height="795" alt="Screenshot 2026-02-09 alle 12 43 59" src="https://github.com/user-attachments/assets/7349cc28-f236-4bb2-bfd7-9f53a96efa74" />

### Visualizing analytics with Grafana:
In Grafana WebUI, the dashboard previously imported will provide the following four analytics, all related to the time range 2025-11-10 00:00:00 - 2025-11-10 23:59:00:<br>
<img width="1610" height="955" alt="Grafana dashb 1" src="https://github.com/user-attachments/assets/17d625e8-c2cf-400a-8843-90a8281193ff" />

Let's examine each analytic in detail.

**1. Overall measures in time range (time series):** this analytic visualizes all the measures included in the given time range.
<img width="1908" height="943" alt="Grafana dashb 2" src="https://github.com/user-attachments/assets/b2cb1ace-daa9-4395-8363-4b8e21f8b841" />
InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc1-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
```
**2. Focus on measures: temperature (time series):** this analytic visualizes the temperature trend (for a specific sensor) in the given time range, with evidence of outliers.
InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc1-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
|> filter(fn: (r) => r["deviceId"] == "sensor-R")
|> filter(fn: (r) => r["_field"] == "pressure")
|> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
|> yield(name: "pression_phase_R")
```
**3. Focus on measures: pressure (time series):** this analytic visualizes the temperature trend (for a specific sensor) in the given time range, with evidence of outliers.
InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc1-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
// Specific filter for sensor R
|> filter(fn: (r) => r["deviceId"] == "sensor-R")
|> filter(fn: (r) => r["_field"] == "temperature")
|> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
|> yield(name: "temperature_phase_R")
```
**4. Measures per operative status in time range (pie chart):** this analytic monitors status changes in the given time range, with evidence of which sensor is reporting statuses other than normal (‘operational’) and how often.
InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc1-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
|> filter(fn: (r) => r["_field"] == "temperature") 
|> group(columns: ["status"])
|> count()
|> yield(name: "status_count")
```
