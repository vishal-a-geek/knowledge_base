#go #passwords #salt-passwords #rainbow-tables 

# Salt Passwords

The goal when securing passwords is to do it in such a way that even if an attacker gets a copy of our database, they still won’t be able to figure out what each user’s password is. While we don’t ever want to have a data breach and leak our database to hackers, designing our system this way will ensure that even if it does happen our users don’t need to worry about accounts on other websites that share a password also being compromised.

It isn’t possible to take a hash value and reverse it into the original password. Knowing this, we might think we are covered just by hashing passwords, but unfortunately hackers are a pesky bunch and over time they have developed quite a few tricks and techniques. One of those techniques is using **rainbow tables**.

While it isn’t possible to reverse a password hash into its original value, it is possible to take a list of common passwords and to hash each of them. This is called a rainbow table.

Once a rainbow table has been created, it is possible to take the list of generated hash values and to compare them with hash values in a stolen database. The goal here is to look for any passwords that match a value in the rainbow table, then once a match is found the attacker knows what common password was used to generate that hash value.

Let’s look at an example to further clarify. Imagine that a hacker was able to get access to our database, as well as the information about the hashing function and any secrets we are using with it. They might end up with a list of hashed passwords like below:

```bash
id | password
-- | ------------
 1 | c66d8e7d45e7
 2 | eff5364b19f5
 3 | 93435c1a22a9
 4 | b78e5973fae1
 5 | 7c12e834ef44
 6 | 79e1f4ccf82e
 7 | 890b86bf0779
 8 | df5d6e16ac62
 9 | 30fa6a3d24fe
```

The attacker cannot determine any user’s password directly from the hash, but since they also know our hashing function, they could create a list of common passwords that they think users might be using. This could even include randomly generated passwords produced by code the hacker wrote. Below is a shortened list as an example.

```css
password
abc123
secret
none
```

Next, the hacker could use the information about our hashing function to run all of these through code that generates a table that has both the original password and the hash value it produces.

```sql
password | hash
-------- | ------------
password | 93435c1a22a9
abc123   | c66d8e7d45e7
secret   | df5d6e16ac62
none     | 79e1f4ccf82e
```

This is called a rainbow table, and with it a hacker could take the stolen database and compare it with the rainbow table to see if any of the password hashes match. Comparing this to our leaked database they could learn the following user passwords:

```sql
id | password     | cracked_pw
-- | ------------ | ----------
 1 | c66d8e7d45e7 | abc123
 2 | eff5364b19f5 |
 3 | 93435c1a22a9 | password
 4 | b78e5973fae1 |
 5 | 7c12e834ef44 |
 6 | 79e1f4ccf82e | none
 7 | 890b86bf0779 |
 8 | df5d6e16ac62 | secret
 9 | 30fa6a3d24fe |
```

While it didn’t tell the attacker every user’s password, it did give them several matches and they could either try logging in as that user, or they could even try other websites, like PayPal, to see if the user used the same password on multiple sites.

The first challenge for hackers using rainbow tables is that it takes a while to create each rainbow table. Hashing millions or billions of possible passwords is feasible, but it takes time. As a result, this attack works best when it can be applied to a large set of hashed passwords all at once.

That is where salting passwords becomes beneficial. Salting a password is the act of adding random data to each password before hashing it. Every user would get their own unique salt value, so we also store the salt in our database so that we can use it when verifying a user’s password.

For example, when a user signs up they might type in their password of `abc123`. At that time we would generated the random salt value - let’s say `ja08d` - and we would append it to their password giving us the string `abc123-ja08d`. We could then hash this new string, getting the hash value of `64047ee6222f`. We would then store all of this in our database.

```bash
id | password_hash | salt
-- | ------------- | ----------
 1 | 64047ee6222f  | ja08d
```

When the user signs in and gives us a password to authenticate, we know to add the `ja08d` salt to the input before hashing it and comparing it to their hashed password value, so we haven’t lost any functionality, but how does this help us with rainbow table attacks?

Each user should be given their own salt value. This value will be stored in the database, so in the case of a data breach a hacker may have access to the salt values, but by using a unique salt for each password we have effectively made rainbow tables useless. Attackers now need to generate a rainbow table separately for each salt, which is equivalent to just guessing each individual password without the use of a rainbow table.

#### `bcrypt` has salt built in

We will be using `bcrypt` to hash our passwords, and because this algorithm was designed specifically for passwords it actually handles generating salts for us. Looking back at our `abc123` example, rather than having a new column in the database, bcrypt will append the salt to the hashed password giving a result like `64047ee6222f.ja08d`. Later when comparing passwords bcrypt knows to pull the `ja08d` salt out of the hash, allowing us to avoid a whole new column in our database for the salt.

Another related topic is a **password pepper**. Like a password salt, a pepper is random value added to the password before hashing, but rather than being user-specific, a pepper is application specific. The main benefit here is that because a pepper is the same across the entire app, it does not need to be stored in the database and can instead be stored in the application. The hope here is that if an attacker gains access to our database, but not other parts of our application, they will have a harder time cracking passwords.

In practice a pepper isn’t necessary and the rest of our security measures are sufficient, so we won’t be including it in our app, but it is worth knowing what it is in case it comes up in the future.

#### Where do the terms salt and pepper come from?

Salting is a term that has been around in cryptography for a long time, but unfortunately is isn’t 100% clear where it came from.

Some say it was inspired by Roman soldiers who would salt the earth to make it less hospitable during wars. Others claims it stems from [salting a mine](https://en.wikipedia.org/wiki/Salting_\(confidence_trick)), the act of adding gold or silver to an ore sample with the intent to deceive anyone who was considering buying the mine.

Regardless, salting was the first term to be coined, and then the term pepper came about because salt and pepper is a popular duo (at least in the United States).