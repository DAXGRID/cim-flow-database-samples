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
An equipment can be feeded from multiple sources. If so, there will be multiple rows in the flat_feeder_info table, one for each source. It's important to take this into account when joining with the flat_feeder_info tabel. By filtering on the nofeed and multifeed fields, or only looking at seq_no 1, it's possible to ensure that only one row is joined per equipment.

## Example how to query the flat_feeder_info table
Imagine you have an AC line segment with id 8b7faf52-10bc-4ad2-8901-51c470d87037:
```sql 
select * from cimflow.ac_line_segment_ext where mrid = '8b7faf52-10bc-4ad2-8901-51c470d87037'
```
![image](https://github.com/user-attachments/assets/eb28d782-1805-4d94-8fee-29e7aa4a2a0a)

Now, let's try find it in the flat_feeder_info table:
```
select * from cimflow.flat_feeder_info where equipment_mrid = '8b7faf52-10bc-4ad2-8901-51c470d87037' and switch_state_type = 'GIS'
```
![image](https://github.com/user-attachments/assets/f9cae78e-7335-4800-9c25-61374c61be92)

![image](https://github.com/user-attachments/assets/a7213592-4e6c-4fec-9917-2caef6b232e5)

Notice that I put the in the statment "switch_state_type = 'GIS'" in the where clause. It's to prevent getting results from various topology processing datasets.

As you can se we found only one object, which means the cable is not feeded from multiple sources. You can also see that the multifeed field is false. Moreover you can se that the nofeed is also false.

## How to used the flat_feeder_info to join in information about feeding sources

Lets try create a join our cable with some feeder information, such as the name of the secondary substation it is feeder from.

```sql 
select 
  eq.mrid as equipment_mrid,
  secondary_substation.name as feeder_from_secondary_substation
from  
   cimflow.ac_line_segment_ext eq
left outer join
  cimflow.flat_feeder_info fi on fi.equipment_mrid = eq.mrid
left outer join
  cimflow.substation secondary_substation on secondary_substation.mrid = secondary_substation_mrid
where
  fi.switch_state_type = 'GIS' and 
  eq.mrid = '8b7faf52-10bc-4ad2-8901-51c470d87037'
```

![image](https://github.com/user-attachments/assets/469a09a6-9ae5-4ceb-98ce-127c2a3e5acf)

As you can se, the cable is feeder from a secondary substation named '3320'.

Now, let's try join in the primary substation and bay as well.

```sql
select 
  eq.mrid as equipment_mrid,
  secondary_substation.name as secondary_substation,
  primary_substation.name as primary_substation,
  primary_substation_bay.name as primary_substation_bay
from  
   cimflow.ac_line_segment_ext eq
left outer join
  cimflow.flat_feeder_info fi on fi.equipment_mrid = eq.mrid
left outer join
  cimflow.substation secondary_substation on secondary_substation.mrid = secondary_substation_mrid
left outer join
  cimflow.substation primary_substation on primary_substation.mrid = primary_substation_mrid
left outer join
  cimflow.bay_ext primary_substation_bay on primary_substation_bay.mrid = primary_substation_bay_mrid
where
  fi.switch_state_type = 'GIS' and 
  eq.mrid = '8b7faf52-10bc-4ad2-8901-51c470d87037'
```

![image](https://github.com/user-attachments/assets/50931683-aed9-4116-b91e-20dc7f1c36fe)

So this way it's easy to join in information on where a particular cim conducting equipment is feeder from. As you can see from the fields in flat_feeder_info, it's possible to join in information all they way up to the external network injection. Of course that would require that the network model extracted from GIS contains such details.

## Example how to query number of installations grouped by cabinet and secondary substations

```sql
select 
  secondary_substation.name as substation, 
  cabinet.name as cabinet,
  count(*) as installation_count
from
  cimflow.usage_point up
left outer join
  cimflow.energy_consumer ec on ec.mrid = up.equipments
left outer join
  cimflow.flat_feeder_info fi on fi.equipment_mrid = ec.mrid
left outer join
  cimflow.substation secondary_substation on secondary_substation.mrid = secondary_substation_mrid
left outer join
  cimflow.substation cabinet on cabinet.mrid = cable_box_mrid
where
  fi.switch_state_type = 'GIS' and 
  fi.nofeed = false and
  fi.multifeed = false
group by
  secondary_substation.name, 
  cabinet.name
order by
  secondary_substation.name, 
  cabinet.name
```

![image](https://github.com/user-attachments/assets/8fb6aad1-c228-4cdb-81fb-f81379d5fdc4)

As you can see in the result, there are some rows where cabinet is null. It's installationes that are connected directly to a secondary substation and not trough a street cabinet.







