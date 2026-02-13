# Use Case 2: Data center environmental monitoring (small data)

Monitoring of a data center environmental measures expressede in SenML standard.

## Use Case reference:
- Small dataset (100 records)
- IIoT input data in JSON format
- input data directly injected into a NiFi pipeline’s processor 

## Use Case scenario/operation:
Defined by RFC 8428, SenML is a widely used industry standard because it allows multiple measurements to be grouped into a single package, optimising data weight/payload. It uses abbreviated key names to save bytes.<br>
Strength: International standardisation and interoperability between systems from different manufacturers.<br>
Typical use: Industrial gateways that send data to cloud platforms (Azure IoT, AWS IoT, MindSphere).<br>

To simulate data collection at successive time intervals in the SenML standard, the t (Time) field is used.<br>
In SenML, if a bt (Base Time) is present, the value of t in each record is added to the bt. This allows a historical series to be sent in a single JSON packet very efficiently, using small numbers (offsets) instead of repeating entire timestamps.<br>
In this use case, we simulate a reading every 60 seconds (offset of 60, 120, 180, etc.) for a control unit that monitors temperature and humidity.

## IIoT input data shape:
```
[
  {"bn": "urn:dev:mac:001122334455:", "bt": 1734643800, "bu": "Cel"},
  {"n": "temp", "v": 23.5},
  {"n": "hum", "v": 45, "u": "%EL"}
]
```

## IIoT input data description and reference values/range used in the demo:
- **bn (Base Name)**: device identifier (often a URN or a MAC address).
- **bt (Base Time)**: reference timestamp (Unix epoch).
- **bu (Base Unit)**: The default unit of measurement.
- **n (Name)**: The name of the specific measure.
- **v (Value)**: The value recorded
  
***

## Demo configuration:

### Importing the NiFi pipeline:
In NiFi WebUI:
- drag a new Process Group in the canvas<br>
<img width="478" height="126" alt="NiFI-ProcessGroup" src="https://github.com/user-attachments/assets/eb5c908b-a3d1-4901-8ae4-a79b80b49436" /><br>
- click on the right side icon in the Field name:<br> 
<img width="467" height="266" alt="NiFi-Loading" src="https://github.com/user-attachments/assets/973528b6-0a4d-4343-a37f-e62b5d724485" /><br>
- upload file **air5-eda-uc2-pipeline.json** the pipeline will be placed in the canvas:<br>
<img width="346" height="162" alt="NiFi-UC1" src="https://github.com/user-attachments/assets/03aaa9e4-888a-45a2-93d9-91b28ec0c886" /><br>
- double click on the Process Group, the overall detailed pipeline will be shown, with the evidence of all the steps, ready to be activated:<br>
<img width="857" height="702" alt="NiFi-UC2-pipeline" src="https://github.com/user-attachments/assets/5fa73c9b-d95b-4329-be85-5735a1bab78e" />
<br>
Step/processor's configuration and properties are reachable right-clicking on the processor itself and selecting "Configure".

### Creating the MinIO bucket for data:
In MinIO WebUI:
- create a bucket named "air5-eda-uc2-bucket"
  
### Creating the InfluxDB bucket for data:
In InfluxDB WebUI:
- create a bucket named "air5-eda-uc2-data"
  
### Importing the Grafana dashboard:
In Grafana WebUI:
- import file **air5-eda-uc2-dashb-1770387918405.json** using the import feature under New menu on the upper right:<br>
<img width="1913" height="235" alt="Grafana-Import" src="https://github.com/user-attachments/assets/982e31d1-b2c8-4d53-aed3-ad263f49ca8e" />

It is important to highlight that the import process includes both the creation of the dashboard analytics templates and the configuration of the connection to InfluxDB data, as it can be quite under:
<img width="1664" height="550" alt="Grafana - Influx" src="https://github.com/user-attachments/assets/d303eb94-db6d-44e4-8ac6-5f90b7618dc8" />

## NiFi pipeline description:
Going back to NiFi WebUI, let's give a detail to all the pipeline steps:<br>
- step 1:  submitting the source JSON dataset to the application
as anticipated in "Use Case reference" section above, and as you can check going to the Properties tab of processor's configuration and right-clicking on "Custom text" field, the source dataset is directly inserted internally to the pipeline:<br><br>
<img width="1267" height="596" alt="NiFi-UC2-CustomText" src="https://github.com/user-attachments/assets/ad3c0fbf-0a96-43d7-8476-60fdbd30c11e" /><br>

This step leads to two flows: the first one (2a) intends to store original data (for backup or further analysis reasons) into a MinIO storage area, the second one (2b to 4) represents the core application path (from data to analysis)<br><br>
In order to be easily stored into InfluxDB, JSON data are transformed into Line Protocol format - the most suitable data format expected by InfluxDB. As an example, the JSON file in the "IIoT input data shape" section has this Line Protocol representation:<br>
```
environment,sensor=urndevmac001523ac8211 temp=21.8 1734645120000000000
environment,sensor=urndevmac001523ac8211 hum=42.5 1734645120000000000
environment,sensor=urndevmac001523ac8211 temp=21.9 1734645180000000000
environment,sensor=urndevmac001523ac8211 hum=42.6 1734645180000000000
```
where the last value represents the epoch (nanoseconds) conversion of timestamp format.<br>

- step 2a: storing data into the MinIO bucket
- step 2b: transforming input data into Line Protocol format (InfluxDB friendly) by using the following groovy script:
```
import org.apache.commons.io.IOUtils
import java.nio.charset.StandardCharsets
import groovy.json.JsonSlurper

def flowFile = session.get()
if (!flowFile) return

try {
    flowFile = session.write(flowFile, { inputStream, outputStream ->
        def json = new JsonSlurper().parse(inputStream)
        
        // 1. Extract metadata from the first element
        def metadata = json[0]
        def baseName = metadata.bn.replace(":", "") // Clean MAC address
        def baseTime = metadata.bt as Long
        
        def lineProtocolBuffer = new StringBuilder()

        // 2. Iterate on measures (bypassing metadata first element)
        json.eachWithIndex { entry, index ->
            if (index == 0) return // Bypass header
            
            def name = entry.n
            def value = entry.v
            def offset = (entry.t ?: 0) as Long // If t misses, use 0
            def timestamp = (baseTime + offset)*1000000000
            
            // 3. Build InfluxDB row: measurement,tag field timestamp
            // Format: environment,sensor=MAC value=23.5 1734645120
            lineProtocolBuffer.append("environment,sensor=${baseName} ${name}=${value} ${timestamp}\n")
        }
        
        outputStream.write(lineProtocolBuffer.toString().getBytes(StandardCharsets.UTF_8))
    } as StreamCallback)

    // Set Content-Type for InvokeHTTP
    flowFile = session.putAttribute(flowFile, "mime.type", "text/plain")
    session.transfer(flowFile, REL_SUCCESS)
} catch (Exception e) {
    log.error("Error in Groovy transformation", e)
    session.transfer(flowFile, REL_FAILURE)
}
```
- step 3:  writing data into InfluxDB
- step 4:  logging InfluxDB transactions

***

## Running the demo:

### NiFi pipeline activation:
In NiFi WebUI:
1. Activate (right-click on the processor, then select "Start": the red square icon will turn into a green arrow icon) all the processors except step 1 "Submitting IoT dataset use case 1"
2. Activate processor step 1 for 2 seconds then deactivate it (just to trigger data start flowing along the pipeline)
3. Check the value of "Tasks/Time" in the processors step 3 and 4: when they are no longer equal to zero, it means that the pipeline has finished

### Sending IIoT data to MinIO:
In MinIO WebUI:
- the input dataset has been sent to the bucket previously created:
<img width="1653" height="320" alt="MinIO-UC2" src="https://github.com/user-attachments/assets/926734ea-eb8b-4a7e-9a5b-9f0015bc17d2" />

### Storing process data in InfluxDB database:
In InfluxDB WebUI:
- going to Data Explorer environment and selecting from 2024-12-19 22:00:00 to 2024-12-19 23:59:59 as the custom time range, it will be possible to query, visualize and navigate through data stored, by applying all the desired filters to data and then visualizing them clicking on SUBMIT button:
<img width="1827" height="814" alt="Influx-UC2" src="https://github.com/user-attachments/assets/a0a1c8b7-adb8-4462-8206-f9b4b3628fd8" />

### Visualizing analytics with Grafana:
In Grafana WebUI, the dashboard previously imported will provide the following four analytics, all related to the time range 2024-12-19 22:00:00 - 2024-12-19 23:59:59:<br>
<img width="788" height="361" alt="Grafana-UC2-dashb-gen" src="https://github.com/user-attachments/assets/3183c874-fb21-40e2-8300-892921aad5c0" />

Let's examine each analytic in detail.

**1. Overall measures in time range (time series):** this analytic visualizes all the measures included in the given time range.

InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc2-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
```
**2. Temperature last value (stat):** this analytic visualizes the last value of temperature in the given time range.

InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc2-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
|> filter(fn: (r) => r["_field"] == "temp")
|> last()  
```
**3. Hunidity last value (stat):** this analytic visualizes the last value of humidity in the given time range.

InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc2-data")
|> range(start: v.timeRangeStart, stop: v.timeRangeStop)
|> filter(fn: (r) => r["_field"] == "hum")
|> last()
```
**4. Intensity heatmap (values distribution):** this analytic shows a grid where the most intense colour indicates that the sensor detected that specific value most frequently during that time period.

Instead of showing just a line, this panel divides time into buckets and colours the areas based on the frequency of the values.
Grafana configuration:
- Panel: Select the ‘Heatmap’ type.
- Data format: ‘Time series’.
- Colour scheme: Select ‘Oranges’ or “Plasma” to highlight the ‘hot spots’.

InfluxDB query underlying the analytic:
```
from(bucket: "air5-eda-uc2-data")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["n"] == "temp")
  // Let's aggregate in small intervals to create density
  |> aggregateWindow(every: 1m, fn: mean, createEmpty: false)
```
