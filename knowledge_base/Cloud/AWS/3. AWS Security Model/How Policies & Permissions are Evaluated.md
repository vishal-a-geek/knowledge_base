#aws #cloud 
- Policies are Combined
	- A user might have multiple policies attached
	- A user might be a part of multiple user groups
		- Where each user group might have a set of policies attached
	- Thus, while evaluated policy of a user, all these users are attached
- What happens if permissions are clashed?
	- Scenario: A Policy adds missing permissions
		- **Policy A**: Only read EC2
		- **Policy B**: Allow writing (starting, stopping etc.)
		- **Final Set**: Allow both reading and writing EC2
	- Scenario: A Policy clash
		- **Policy A**: Only read EC2
		- **Policy B**: Denies all EC2 access
		- **Final Set**: No EC2 access
			- i.e. "Deny" permission always overrides "Allow"

**When and How are the permissions evaluated?**
- If a user has a policy attached which allows reading EC2, and if the user tries to "List EC2 instances"
	- AWS evaluates permissions attached to the identity involved while sending the request
	- If the permissions list contains the permission to list EC2 instances, then the command is allowed, or else not