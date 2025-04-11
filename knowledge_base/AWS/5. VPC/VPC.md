A Simple/Realistic Setup:
- 1 EC2 Instance running web server
- Connected to internet
- Receives/Sends requests from/to internet
- Another EC2 instance running a database used by the other EC2 instance for data exchange
	- This **should not** accept/receive requests from public internet
	- This **should be** able to send requests to internet
	- There should not be any technical possibility to be able to connect to such instance from public internet

**VPC**: A group of resources (e.g. EC2 instances) inside a customer-managed network
- It is divided into subnets
	- Can have private/public subnets
		- Private subnet: Only allows internal requests
		- Public subnet: Also allows internet requests
	- All instances in a VPC can talk to each other no matter in which subnet (public/private) they are
	- Using subnets, we can control
		- Internet connectivity
		- Region for launches resources/instances
			- A region has multiple Data-centres which are further grouped into AZs
			- While subnetting instances, we can choose AZs of the subnet
				- All instances in a subnet belong to same AZ
	- A subnet becomes public when we add **Internet gateway** to it
	- A private subnet does not have internet gateway
		- To get internet connectivity within private subnet (for e.g. to download patches), by making sure only outgoing requests to the internet are allowed from within the private subnet, and no incoming request can reach the private subnet, use **NAT(Network Address Translation) gateway**
			- NAT gives private subnet an indirect access through the internet gateway, but only for outgoing requests
			- NAT costs extra money (is not covered by free tier)
- There is a **default VPC** per region provided by AWS
- VPC is a regional service
	- Thus if we want to launch an EC2 instance within a VPC, we can only launch it in a region in which the VPC is present
- We can mention # AZs
- We can mention # public/private subnet
- **Routing tables** are created one per region and one globally
	- Routing tables are created automatically
- VPC
	![[Pasted image 20250401071642.png]]
- Public subnet
	![[Pasted image 20250401071740.png]]
- Private subnet
	![[Pasted image 20250401071835.png]]

## CIDR IP Ranges
- Every VPC has an IP CIDR block assigned (range of IPs)
	- Subnets get parts of the VPC CIDR block assigned
	- EC2 instances receive auto-assigned private IPs from among that CIDR block
- 172.31.0.0/16 -> represents a range of IP addresses
- **/16** denotes how many bits are fixed
	- 172.31.0.0 -> 172.31.255.255
- 172.31.0.0/24 -> 24 bits are fixed
	- 172.31.0.0 -> 172.31.0.255
- 0.0.0.0/0 -> no bits are fixed
	- All possible IP addresses

## Public and Elastic IP Addresses
- **Public IP Address**: automatically assigned by AWS and removed/unassigned when the instance is stopped/terminated, and when the instance is up again, AWS might assign a different public IP addresses. Thus it is not necessary for the instance to get same IP address every time.
- **Elastic IP Address** : managed and assigned by us and same IP is assigned to instance every time
	- We can only have few Elastic IPs per region and AWS account
	- Unused elastic IPs incur charge
	- There often better alternatives for Elastic IPs
		- Use DNS rather for exposing applications / websites to the world

## Security Groups & NACLs & Private/Public Subnets
 - **Security Group**
	 - Firewall for a single EC2 instance
	 - Checks incoming/outgoing requests and conditionally blocks or allows them
	 - If request is allowed, the response is always allowed
	 - Can be reused across multiple EC2 instances
 - **NACL(Network Access Control List)**
	 - Firewall for an entire subnets
	 - Less fine-grained control than security groups
	 - For response to be allowed, a separate rule is needed other than rule for request.
		 - Just because request was allowed to enter a subnet, does not mean response from the subnet will also be allowed back
	- Can be associated with multiple subnets
 - **Private/Public Subnets**
	 - Defines technical connectivity
		 - Both security groups and NACLs talk about **firewall**
			 -  i.e. here the request **reaches** the instance/subnet and then the firewall decides what to do with it
		- the request can't even **reach** the subnet/instance from internet if the subnet is private
	-  No internet access possible without **Internet Gateway(IG)**
		- i.e. no incoming/outgoing requests without **IG** to a public subnet
		- no outgoing requests without **NAT** to a private subnet
	- Can't control the kind of requests (HTTP or SSH) to be allowed/blocked
	- Multiple instances can be in same subnet

## VPC Peering & Transit Gateways
- **VPC Peering**: Connecting 2 VPCs
-  **Transit Gateways**: Connecting more than 2 VPCs 

## VPC Endpoints & AWS PrivateLink
- While connecting another AWS service (like S3) from within VPC, the request can be sent via different ways
	- Via internet
		- Requires Internet gateway and or NAT gateway
	- AWS network (or AWS PrivateLink)
		- Using VPC Endpoints
		- No internet connectivity required

