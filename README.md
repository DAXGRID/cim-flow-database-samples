# CIM Flow Database
Documentation and example queries demonstrating the utilization and querying of the CIM database

## Understanding the flat_feeder_info table
The flat_feeder_info table contains one or more records for all conducting equipments in the network.

![image](https://github.com/user-attachments/assets/91b3847b-cf8b-4883-b6db-e2130ef85096)

Firstly it's important to understand the three fields marked above.

### switch_status_type
The switch_status_type will contain the values "GIS", "Normal" or "Actual". 

The rows having the value 'GIS', is the default feeder dataset created by the topology processor. The data is based on the switch.normalOpen values in the equipment model extracted from GIS. Hence the name GIS.

The rows having the value 'Normal', is a feeder dataset based on a normal switcing state dataset extracted from some external system - e.g. a SCADA system. That is, the switch normalOpen value in the equipment model is not used.

Same with rows having the value 'Actual'. This is a feeder dataset based on a actual switcing state dataset extracted from some external system. 

### equipment_mrid
This is the id of the conducting equipment the feeder information belongs to. In other words this is the key to be used in joins, when you want feeder information joined to objects in your queries.

### seq_no
An equipment can be feeded from multiple sources. If so, there will be multiple rows in the flat_feeder_info table, one for each source. It's important to take this into account when joining with the flat_feeder_info tabel. By filtering on the no_feed and multi_feeded fields, or only looking at seq_no 1, it's possible to ensure that only one row is joined per equipment.

## Example how to query the flat_feeder_info table
Imagine you have an AC line segment with id 8b7faf52-10bc-4ad2-8901-51c470d87037:
```
select * from cimflow.ac_line_segment_ext where mrid = '8b7faf52-10bc-4ad2-8901-51c470d87037'
```
![image](https://github.com/user-attachments/assets/eb28d782-1805-4d94-8fee-29e7aa4a2a0a)

Now, let's try find it in the flat_feeder_info table:
```
select * from cimflow.flat_feeder_info where equipment_mrid = '8b7faf52-10bc-4ad2-8901-51c470d87037' and switch_state_type = 'GIS'
```
![image](https://github.com/user-attachments/assets/f9cae78e-7335-4800-9c25-61374c61be92)

Notice that I put the in the statment "switch_state_type = 'GIS'" in the where clause. It's to prevent getting results from various topology processing datasets.

As you can se we found only one object, which means the cable is not multi feeded. You can also see that the multifeed field is false. Moreover you can se that the nofeed is also false.

