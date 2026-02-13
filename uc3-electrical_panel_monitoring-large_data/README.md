# Use Case 3: Electrical panel monitoring (large data)

Monitoring of a critical electrical panel (three-phase electrical system)

## Use Case reference:
- Large dataset (4000 records)
- IIoT input data in JSON format
- input data read by NiFi pipeline’s processor from a file allocated in a shared network storage area

## Use Case scenario/operation:
Let's replicate exactly the same Use Case 1 scenario but playing with a large instead of a small dataset (and ingesting data from an external file instead of injecting it directly into the NiFi processor)

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
<br><br>
The input data file "IIoT-shape-typeA-UC3.json" has been generated in this way:
- 1000 record for each of the four sensors
- "timestamp" starts from 2025-11-10 00:00:00 and increments by 1 sec for every new record
- "temperature" is a random value between 9.5 and 14.0 and must be in the range 10.0 - 13.0 for the 95% of records generated
- "humidity" is a random value between 4.5 and 8.0 and must be in the range 5.0 - 7.4 for the 95% of records generated
- "pressure" is a random value between 10.0 and 20.0 
- "battery" is a random value between 3.0 and 3.8 and must be in the range 3.3 - 3.6 for the 95% of records generated
- "status" is the fixed string:
	- "warning" if temperature is not in the range 10.0 - 13.0 or humidity is not in range 5.0 - 7.4 
	- "critical" if battery is higher than 3.6 and (temperature is not in the range 10.0 - 13.0 or humidity is not in range 5.0 - 7.4)
	- "out-of-range" if battery is lower than 3.3
	- "operational" in all the other cases

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
<img width="1743" height="691" alt="NiFI-UC3-pipeline" src="https://github.com/user-attachments/assets/c03886ac-4a93-4a45-a015-d50136d4d157" />
<br>
Step/processor's configuration and properties are reachable right-clicking on the processor itself and selecting "Configure".

### Preparing the input file for the NiFi pipeline:
Thanks to the following directive in the docker-compose.yml file:<br>
<img width="307" height="32" alt="NiFi-UC3-file" src="https://github.com/user-attachments/assets/789d4710-e81a-49c6-b834-edba7cd16c60" /><br>
and having created "source-data" directory for input data (see Deployment Steps - Step 1), let's copy "IIoT-shape-typeA-UC3.json" file in that directory.<br>
In genaral, you can copy the data related to your specific use case: just make sure it complies to "IIoT input data shape" format and it is included in a file with ".json" suffix.

### Creating the MinIO bucket for data:
In MinIO WebUI:
- create a bucket named "air5-eda-uc3-bucket"
  
### Creating the InfluxDB bucket for data:
In InfluxDB WebUI:
- create a bucket named "air5-eda-uc3-data"
  
### Importing the Grafana dashboard:
In Grafana WebUI:
- import file **air5-eda-uc3-dashb-1770387931287.json** using the import feature under New menu on the upper right:<br>
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
```
