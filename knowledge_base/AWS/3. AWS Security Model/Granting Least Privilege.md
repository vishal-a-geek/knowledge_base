An important security concept is the idea of "**Granting Least Privilege**".

It means that you should never grant more permissions (to anyone) than needed.

By default, new roles or users have no permissions. They can't do anything. You can then add more permissions to them by assigning policies. But instead of adding the "Full Admin" policy to all your users, you should really only add the policies needed by a given user.

#### Manage Programmatic Access Keys

Related to the concept of "Least Privilege", you should also ensure that IAM users only have programmatic access keys (for the CLI & SDKs) if they need those keys. Otherwise, don't create the keys.

**For the root account, you should also delete the programmatic access keys as a best practice.** As mentioned in earlier lectures, the root account user should basically never be used because it has unlimited access rights.