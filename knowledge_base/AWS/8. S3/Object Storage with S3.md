## Understanding object storage
- We don't focus/care on underlying hard-drive or the file system being used
- Just focus on the file itself
	- All we know is we got a file (image/pdf) and we want to store that file in the cloud
		- Either because it's getting used by an application
		- or it was produced by a workload on EC2
		- or store personal/business files which are saved on cloud rather than on local machine
- We think of storing **any kind** of files with almost any kind of size
	- The limit on size on S3 per file is 5TB
	- No limit on number of files we can have
- Features:
	- File storage
		- Files can be uploaded via console, CLI, HTTP API, SDK
		- Unlike EBS or EFS, S3 is not a service that has to be attached to another service
	- Buckets
		- Files are organised into buckets (folders)
			- Though we cannot have nested buckets
		- Files are stored in buckets
		- S3 does not provide a file system where u could create subfolders.
			- Can create multiple buckets
		- Instead all files are stored in buckets
		- Buckets are **regional**
		- Must have unique name across all AWS accounts in the entire world
	- No file system present
		- We have it under the hood, but as a user of S3, we don't care or manage any file system
		- It looks like flat structures or no directories
		- Prefixes help organising files
	- Detailed Permissions System
		- bucket and/or file level access control
		- bucket policies (a recommended approach)
		- Access control list (not recommended) 


## Storage Classes
- Different Storage classes 
	- ![[Pasted image 20250402225015.png]]
### Different file access patterns
- Frequent access
	- Accessed very frequently (every second/minute)
	- Instant access required
	- Highest flexibility with no cost saving
	- *Standard* storage class
- Infrequent access
	- Accessed infrequently or only from time to time
	- Instant access possible
	- Come with retrieval cost and less storage cost
	- *One Zone-IA* storage class
- Archive
	- Almost never accessed
	- Instant access not always possible
	- High cost savings but less flexible
	- *Glacier* storage classes 

### Intelligent Tiering
- AWS will analyse access patterns of a file and will automatically move it in fitting storage class category
## S3 Features
- Versioning
	- Store multiple versions of a file
	- Cost extra money
- Lifecycle management
	- Transition files between storage classes
		- AWS can automatically move files to infrequent storage class after say 30 days
- Inventory & Analytics
	- Understand stored files and data
- Object lock
	- Prevent object deletion / changes
- Cross bucket Replication
	- Auto replicate objects cross bucket
	- Even if a bucket is compromised or deleted, the files are still available in another bucket
	- The replication bucket could either be in same region or even in cross region
- Data protection and Encryption
	- Automatically encrypt stored data
- Static website hosting
	- Upload and host static website files 

## S3 vs EBS vs EFS
- EBS
	- Attachable hard drives
	- EC2 only
	- Automatically scaling and Multi attach
		- Difficult to use these features though
- EFS
	- Attachable file systems
	- Not limited to EC2
	- Automatic scaling and multi attach are key features
- S3
	- Independent object storage
	- Access with/without services
	- Just worry about the object we store
		- Need not bother about any file system or hard drive
	- Unlimited scaling built-in