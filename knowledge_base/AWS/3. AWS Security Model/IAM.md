IAM: Identity & Access Management
- Identities
	- **Who** is allowed to something
	- The **entity** for which permissions are being controlled
	- Users, User Groups & Roles
- Access Management
	- **What** permissions are allowed to an entity or identity
	- The permissions that are granted (or not granted) to an entity
	- **Permissions**, managed via **Policies**
	- Policies group permissions together
		- Polices are documents containing permission descriptions


**Identity: Users, User Groups & Roles**
- Users
	- Entities created for real human people to share access to an account
	- Every User gets his / her own credentials to login
	- After creating a user, 
		- we get a link to sign in with the created user instead of signing in as a root user
			- E.g.: https://478234038964.signin.aws.amazon.com/console
		- Access Key and Secret access key
			- Needed for configuring AWS CLI and AWS SDKs
			- For programmatic or command line driven access
			- Needed for sending authenticated requests or commands to AWS APIs
			- Access keys are long term credentials, unlike IAM Roles which are short term or temporary credentials
			- ![[Pasted image 20250323213536.png]]
- User Groups
	- Group of users sharing same set of permissions
	- Avoid unnecessary permission copying or tedious per-user access management
- Roles
	- Typically used by services
	-  Idea:
		- We often have **services** that need to **work** with **each other**
		- E.g. start or use another service
			- an EC2 service trying to use S3 for sending or storing a file

**Roles**
- Many AWS Services create roles automatically for us not having to go through this process yourself