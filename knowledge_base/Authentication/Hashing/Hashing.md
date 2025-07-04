#hashing #authentication #cryptography 

- Basic idea of hashing function:
	- any data of any size or type, can be converted to a fixed sized text representation
	- ![[Hashing_basic_idea]]
	- fixed sized representation is called hash or digest


- Properties of **hashing function**:
	- **No collisions**: hash/digest is unique to that data
		- Same data gives same hash every time
		- two different data cannot give same hash
	- **Data cannot be recovered from the hash**
		- If original data is lost and we only are left with hash, we cannot do anything with hash or we cannot get back the original data
- Popular hashing algorithms:
	- SHA-512
		- Creates a 512 bit long hash/digest
	- SHA-256
		- Creates a 256 bit long hash/digest
	- MD5

- Use cases:
	- Passwords
		- Passwords are usually stored as hash in identity management systems
		- ![[Password]]
	- File hash
		- Helpful to figure out if the entire file has been downloaded correctly 
		- to make sure that nobody has tampered with it's contents
		- ![[File hashing.png]]