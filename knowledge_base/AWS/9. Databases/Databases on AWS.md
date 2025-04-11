- Different Types of DBs
	- SQL - Relational & Fixed Schema
	- NoSQL - Non-Relational & No Schema
	- Different data or workload requirements favour different database types
	- AWS allows to run and use all DB types
- Different Hosting Options
	- Self host vs managed
	- you can install + operate yourself (e.g. on EC2 instances) or let AWS do that

## Self Hosted Vs Managed DB
- Self hosted
	- Install and operate DB software manually on EC2 instances
	- Have full control but also full responsibility
	- Important duties: Keep software patched, managed backups etc.
	- Have in-house DB experts in the team
- Managed
	- Don't have in-house DB experts in the team
	- Let AWS manage setup and DB operations
	- Key services: RDS, DynamoDB
	- Less control but also less responsibility
	- We can define general rules but AWS does the heavy lifting
	- DB softwares running on top of EC2 instances, though the instances are hidden to us and managed by AWS


## Relation Database Service (RDS)
- Choose DB Engine 
	- MySQL, PostgreSQL
	- Choose auto-update settings
- Choose Hardware and Network configuration
	- Choose instance class (hardware profile)
	- VPC, subnet & security group
- Configure DB Server
	- Login credentials
	- Replication (for high availability and performance)
		- AWS manages maintaining automatic read replicas
	- Monitoring settings
	- Encryption and backup settings

## Databases & VPCs
- We often have public and private subnets in a VPC
- Databases are typically present in a private subnet of the VPC
- VPC
	- Public subnet
		- EC2
	- Private subnet
		- Databases (self-hosted or managed)

## Caching Data with ElastiCache
- Fully managed in-memory caching database
- Managed Redis/Memcached service

## DynamoDB
- NoSQL key-value DB
- We only create tables, no DB is visible to us
	- Set name and keys
	- Set expected read/write capacities
- Manage data
	- Read / Write using AWS API/CLI/SDK
	- No general query language available to query data 
- Features:
	- Streams
		- Time-ordered series of db item changes
		- Subscribe process item changes
	- Global Tables
		- Fully managed multi-region db with automatic replication, thus high availability
		- Performant
	- DAX
		- Database acceleration
		- Managed in-memory invisible cache in DynamoDB
		- Accelerates DB queries
		- Low latency
## Other DB services
- MemoryDB
	- Persistent in-memory storage
- DocumentDB
	- Document database
	- ~ MongoDB
- Keyspaces
	- Wide column database (flexible column formats)
- Neptune
	- Graph DB
- Timestream
	- Time series DB
- Quantum Ledger DB (QLDB)
	- Immutable log of data changes (blockchain)

## Saving data with AWS Backup
- RDS automatically creates Backups continuously
- Centralised backup management
	- Create plan: Set frequency, retention period, timeframe, destination
	- Manage backups: Access and restore backups