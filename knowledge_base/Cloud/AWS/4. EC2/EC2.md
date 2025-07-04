#aws #cloud #ec2 
- An EC2 instance is a slice of physical machine fully isolated from other instances, with it's own dedicated hardware
- We rent those instances of the virtual server while creating an EC2 instance 

- EC2 Image Builder Service
	- Can be used to build images or image pipelines
	- to build custom AMIs

- EC2 instance types
	- Grouped into instance type [families](https://aws.amazon.com/ec2/instance-types/)

## EC2 Pricing
- **On Demand** Instances
- **Spot** Instances Payment model
	- Spare instances, can be reclaimed by AWS anytime
	- Can be useful for tasks which could be interrupted and restarted on interrupt (unlike something like a web server which needs to be up and running always)
- Savings Plan
	- Pay in advance
	- Discount over on-demain pricing
	- Ideal for long-term commit and save money
- Reserved instances
	- Pay in advance for chosen instance types
	- Discounts over on-demand pricing
	- Not very flexible