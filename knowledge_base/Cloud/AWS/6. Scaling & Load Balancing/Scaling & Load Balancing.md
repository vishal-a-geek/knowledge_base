#aws #cloud 
- Need for flexibility
	- Capacity can by dynamically adjusted based on incoming load
## Scaling with AWS Auto Scaling
- EC2 Auto Scaling
	- Automatically Adds / removes EC2 instances based on conditions
- Elastic Load Balancer (ELB)
	- Distribute Load evenly across available multiple

## Elastic Load Balancer & Auto Scaling Groups
#load_balancer
- Application Load Balancer (ALB)
	- Feature-rich
	- Broad variety of request forwarding conditions & rules
	- Capable of SSL termination
		- Load balancer can act as endpoint for incoming HTTPS requests, which would then decrypt the HTTPS request, forwarding the unencrypted request to the instances behind the load balancer
	- No Fixed IP Address
		- can register custom domain to Application LB
	- Reduce app complexity
	- Good for most HTTP apps
	- ALB is limited to HTTP(S) traffic.
- Network Load Balancer (NLB)
	- Very lean
	- less features
	- limited configuration
	- Capable of SSL termination
	- Fixed IP address by AWS
	- Good for non-HTTP traffic
	- Useful for non-HTTP services
- ELB can be attached to Auto-Scaling Groups
- VPC and subnets chosen for ELB and Auto Scaling Groups should be same for the ELB to be capable of forwarding incoming requests to instances in those subnets in that VPC 
- Auto scaling groups can have group sizes:
	- Desired capacity (current capacity)
	- Min capacity
	- Max capacity
- Scaling policies based on:
	- Average CPU Utilisation (%)
	- Average network in (bytes)
	- Average network out (bytes)
	- ALB request count per target
- ALB & NLB don't automatically distribute traffic across all instances that exist in a VPC. Instead, we have to define which instances should be covered (via **target groups**).