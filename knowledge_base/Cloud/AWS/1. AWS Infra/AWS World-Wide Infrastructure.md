#aws #cloud
AWS Operates data centres all over the world
**AWS Region**
- One region can contain multiple data centres
- Regions are physically **isolated** from each other
- Can choose which region to run a service on
	- Some services are global, for which we cannot choose a region
- Reasons for picking a certain region:
	- Different pricing
	- Service availability
		- Not all services are available in all regions
	- Legal Reasons
	- High Availability and Latency

**Regions & Availability Zones (AZs)**
- A region has multiple data centres
- Data centres in a region are further grouped into AZs
- One region contains **at least ==three==** AZs
- One AZ contains **==at least one==** data centre 
- AZs are also separated from each other (less far than across regions)
- All AZs have separate power supply and buildings
- Thus if a catastrophe hits a region, one or two AZs or one or two Data centres might be affected, and need not be the entire region or all AZs of the region
- ![[Pasted image 20250322214622.png]]

- AWS regions are connected over a network owned and operated by AWS
	- AWS owns and operates it's own cables
	- Secure fast connectivity
	- Need not use public internet
	- Thus, better to use AWS network