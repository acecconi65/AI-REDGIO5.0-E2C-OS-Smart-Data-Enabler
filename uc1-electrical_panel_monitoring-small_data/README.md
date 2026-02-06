# Use Case 1: Electrical panel monitoring (small data)

Monitoring of a critical electrical panel (three-phase electrical system)

## Use Case reference:
- Small dataset (100 records)
- IIoT input data in JSON format
- input data directly injected into a NiFi pipeline’s processor 

## Use Case scenario/operation:
Four IoT sensors – one for each phase R, S, T and Neutral - positioned inside a critical high-voltage substation wide electrical panel, a cooled environment for which it is fundamental to check data in order to highlight anomalies.<br>
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
- operation: representing the process operation, always equal to "EP-G02-monitoring"
- device ID: one of the four EP sensors’ serial code, can be one of “sensor-R" o “sensor-S" o “sensor-T" o “sensor-N"
- timestamp: time of measurement collection, can be a value between 00:00 and 23:59 on 2025-11-10
- temperature: the optimal operational range is 10.0-13.0 [°C], indicating a cooled environment
- humidity: the optimal operational range is 5.0-7.4 mbar for low (10.0 °C) temperatures, that is to be maintained to prevent electric arcs or partial discharges due to condensation and prevent both condensation and electrostatic discharge - at 10 °C, the air is ‘saturated’ (i.e. it reaches 100% humidity and begins to form condensation) when the water vapour pressure reaches approximately 12.3 mbar.
- pressure: the optimal operational range is 10.0-20.0 mbar – in highly critical electrical cabinets, the interior of the cabinet is maintained at a pressure slightly higher than the external pressure using nitrogen or filtered dry air - if the pressure drops, it means that the seals on the panel are worn or that a door has been left open - maintaining positive pressure prevents conductive dust, corrosive vapours or external moisture from entering, drastically reducing the risk of electric arcs.
- battery: the optimal operational range is 3.3-3.6 V for low (10.0 °C) temperatures – values lower than 3.0 V have to be considered as alarm, requiring immediate technical intervention
- (device) status:  describing normal or alert/critical situations related to the values detected (voltage, current, terminal temperature):
  - “operational”: normal 
  - “warning”: the value is outside the optimal range, but the system is still safe
  - “critical”: an imminent danger threshold has been exceeded
  - “out of range”: the sensor is detecting a value that exceeds its physical measurement capabilities (often indicating a transducer failure)

***

## Demo configuration:

### Importing the NiFi pipeline:
In NiFi UI:
- drag a new Process Group in the canvas:
<img width="478" height="126" alt="NiFI-ProcessGroup" src="https://github.com/user-attachments/assets/eb5c908b-a3d1-4901-8ae4-a79b80b49436" />
- clic on the right side icon in the Field name: 
<img width="467" height="266" alt="NiFi-Loading" src="https://github.com/user-attachments/assets/973528b6-0a4d-4343-a37f-e62b5d724485" />
- upload file air5-eda-uc1-pipeline.json
  
- the pipeline will be placed in the canvas:
  

-

### Creating the MinIO bucket for data:

### Creating the InfluxDB bucket for data:

### Creating the Grafana dashboard:


## NiFi pipeline description:

***

## Running the demo:

### NiFi pipeline activation:

### Sending IIoT data to MinIO:

### Storing process data in InfluxDB database:

### Visualizing analytics with Grafana:
