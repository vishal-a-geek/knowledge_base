#aws #cloud #aws_s3 s3
## Why File Storage?
- More often, workloads generate some kind of data which needs to be stored in files
	- User uploads
	- Invoices
	- Images (Transformed)
	- Reports
	- Archiving
		- Accounting
		- Legal documents
## Different Kinds of File Storage
- Block Storage (EBS)
	- Unformatted hard drive
	- Format (and structure) before using
	- Create custom structure & store any files
	- Virtual hard-drive to server
	- AWS offers EBS: Elastic Block-Store Service which allows to attach virtual hard-drives only to EC2 instances
		- EBS only focuses on EC2
- Object Storage
	- No information about underlying system storing it
	- Store & retrieve files of any kind of size as needed
	- No custom structure can be created
	- AWS offers S3: Simple Storage Service
- File System
	- A network file system
	- Don't care about hard-drive
	- A pre-formatted & configured file system
	- Create custom structure & store any files
	- AWS offers EFS & FSx

## EBS Volumes
- While launching an EC2 instance, we can automatically add EBS volumes to that instance, i.e. attaching hard-drive(s) to that instance
- Once the instance starts, we have to **structure the** **volume manually**
- We can even add volumes after the instance starts
	- Then, we have to format and mount the volumes manually
- **EC2 exclusive** service
- Features:
	- Can choose SSD vs HDD 
	- Elastic volumes: volumes can be scaled dynamically (elastic)
	- Snapshots: 
		- Take snapshots of EC2 instances and store them on EBS volumes
		- Use that snapshot to restore the data in the EC2 instance for a future EC2 instance
		- Useful from not loosing data if we terminate an EC2 instance
		- Also can take snapshots of volumes
		- On instance termination, whilst the root volume typically is deleted together with the instance, other volumes survive instance termination unless configured otherwise.
	- Multi-attach
		- By default, while creating a new EBS volume, we add to single instance
		- We can attach volumes to multiple instances
			- Supported on some instances only
- Path: AWS Management Console / EC2 / EBS
	- ![[Pasted image 20250402193754.png]]
- EBS is a regional service
	- Volumes are created in specific regions
	- They can be attached to instances in the same region

## EC2 Instance Storage & EBS
- EBS allows to add storage to EC2 instances
- EC2 Instances have some "base storage"
	- Contains OS, base software dictated by AMI
- EC2 Instance storage is an alternative to EBS volume
	- Here, the hard-drive is part of the **physical machine** or **rack** in the data centre
- EBS Volume approach is default approach for adding storage to EC2 instances
	- EC2 instance storage alternative exists for some AMIs
	- While picking an AMI, we also choose where the base software will be installed on
		- Either EBS Volume (most common default)
		- or EC2 Instance storage
	- This can be found by looking at the "**Root device type**" while picking an AMI ![[Pasted image 20250402195320.png]]

## Elastic File System Service
- We also get the hard-drive here but we need not care because we need not or cannot choose the hard drive size or anything, 
	- instead we get a scalable pre-formatted file system (which of course uses hard-drives under the hood)
- Can be structured as we need (add directories/sub-directories)
	- via command line or code
- Cannot format the underlying hard drive
- Can be attached to/accessed from multiple services
	- EBS is EC2 exclusive service
	- EFS is not exclusive to EC2 alone
- EFS is a regional service and needs to know the specific VPC and subnet, while creating a file system using EFS ![[Pasted image 20250402201622.png]]

## EBS Vs EFS
- EBS
	- Unformatted hard drive
	- Less automatic scaling, more manual work
	- Multi-attach is possible on some instances
	- EC2-exclusive
- EFS
	- Pre formatted file system
		- Need not worry about underlying hard drive
	- Scales automatically
	- Multi-attach is a core feature
	- Can be used with multiple services (like also with AWS Lambda)


## FSx Lustre & Others
- Sometimes alternatives to EFS
- It is a file system optimised for high-performance file access workloads
- No feature equality with EFS