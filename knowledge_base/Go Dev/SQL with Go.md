#go #sql #database/sql #pgx #sql-injection 

### Connecting database from Go
In order to connect to a database, we need 2 things:
- **Library** to interact with database using `sql`
	- the go standard library has a `database/sql` package which allows to interact with SQL, but it doesn't provide drivers for every single database
	- `database/sql` provides a **generic** interface around SQL databases
	- `database/sql` package must be used in conjunction with a database driver.
- **Driver** - a postgres driver
	- There are many. E.g. `github.com/jackc/pgx`, `gorm.io/gorm`

Then, we need to register the driver with `database/sql` package using `sql.Register`. 
This is done by the `init` method of the driver package we import using `_ "github.com/jackc/pgx/v4/stdlib"`.
`github.com/jackc/pgx/v4/stdlib` invokes `sql.Register` method with the driver name as `pgx`.
Thus, we just need to open a connection with the database or establish a database connection.

```go
package main

import (
	_ "github.com/jackc/pgx/v4/stdlib"
	"database/sql"
)

func main() {
	db, err := sql.Open(
		"pgx",
		"host=localhost port=8000 user=baloo password=junglebook dbname=lenslocked sslmode=disable",
	)
	if err != nil {
		panic(err)
	}
	defer db.Close()
}
```

- `sql.DB.Close` closes the database connection and prevents new queries from starting.
- `sql.DB` is a database handle **representing a pool of zero or more underlying connections**. It's safe for concurrent use by multiple goroutines.
- It is rare to Close a `sql.DB`, as the `sql.DB` handle is meant to be long-lived and shared between many goroutines.
	- What typically happens is we will open up a database connection when our application starts and then we'll keep using that database connection for all web requests that are coming in 
	- and it's not until our server actually shuts down that we'll need to close the database connection
- Whenever we run `sql.Open` code, it creates a database connection to this database, but doesn't tell if the database server is up and running
	- This is because there is no real reason to actually communicate with a server (db server) until somebody starts trying to write some sort of SQL
	- Server could always go down after we open the database connection, and then later the requests would start failing 
- To make sure db is running, we need to communicate
```go
package main

import (
	_ "github.com/jackc/pgx/v4/stdlib"
	"database/sql"
)

func main() {
	db, err := sql.Open(
		"pgx",
		"host=localhost port=8000 user=baloo password=junglebook dbname=lenslocked sslmode=disable",
	)
	if err != nil {
		panic(err)
	}
	defer db.Close()
}
```

- Having hardcoded database connection string is not recommended with passwords directly being used.
- We can approach this by having a `Config` go `type struct` for creating a struct and the goal with this is that we could then load a different version of that `Config` however we want to, whether it's reading from a `yaml` file or `json` file or `env` variables

```go
package main

import (
	_ "github.com/jackc/pgx/v4/stdlib"
	"database/sql"
)

type PostgresConfig struct {
	Host string
	Port string
	User string
	Password string
	Database string
	SSLMode string
}

func (cfg PostgresConfig) String() string {
	return fmt.Sprintf(
		"host=%s port=%s user=%s password=%s dbname=%s sslmode=%s", 
		cfg.Host, cfg.Port, cfg.User, cfg.Password, cfg.Database, cfg.SSLMode,
	)
}

func main() {
	cfg := PostgresConfig{
		Host: "localhost",
		Port: "8000",
		User: "baloo",
		Password: "junglebook",
		Database: "lenslocked",
		SSLMode: "disable",	
	}
	db, err := sql.Open("pgx", cfg.String())
	if err != nil {
		panic(err)
	}
	defer db.Close()
}
```

### Executing SQL
There are typically 3 ways
- `sql.Query(query string, args ...interface{}) (*Row, error)`
	- use when we expect to return 0 or more `Rows`
- `sql.QueryRow(query string, args ...interface{}) *Rows`
	- use when we expect to return just single `Row`
	- When the SQL query returns no rows, `row.Scan()` will return an error `sql.ErrNoRows`
- `sql.Exec(query string, args ...interface{}) (Result, error)`
	- executes without returning any rows
	- instead returns `sql.Result` type which has meta data about the query result like
		- `LastInsertId()` - doesn't work with all databases (doesn't work with PostgreSQL)
		- `RowsAffected()`

#### Creating Table
```go

_, err = db.Exec(`
	CREATE TABLE IF NOT EXISTS users ( 
		id SERIAL PRIMARY KEY, 
		name TEXT, 
		email TEXT NOT NULL 
	); 
	
	CREATE TABLE IF NOT EXISTS orders (
		id SERIAL PRIMARY KEY,
		user_id INT NOT NULL,
		amount INT,
		description TEXT
	);
`)
if err != nil {
	panic(err)
}

fmt.Println("Tables created")
```

#### Inserting Rows
```go
name := "Vishal Govind"
email := "demo@user.com"
_, err = db.Exec(`
	INSERT INTO users (name, email)
	VALUES ($1, $2);
`, name, email)
if err != nil {
	panic(err)
}
fmt.Println("User created.")
```

##### Why it's important to use `$1` and `$2` rather than trying to build the string all by ourself using something like `fmt.Sprintf`?
#sql-injection 
- SQL Injection is a security vulnerability that basically let's the attacker execute some arbitrary SQL on our database without we wanting them to
- It's not SQL written by developers. It's something attackers wrote and it leads to that happening
- Variation of code injection
```go
name := "', ''); DROP TABLE users; --"
email := "vishal@govind.io"

query := fmt.Sprintf(`
	INSERT INTO users (name, email)
	VALUES (%s, %s);
`, name, email)

fmt.Println(query)

_, err = db.Exec(query)
if err != nil {
	return panic(err)
}
```

This will print:
```sql
INSERT INTO users (name, email)
VALUES ('', ''); DROP TABLE users; --', 'vishal@goving.io');

-- then if we try to fetch users:
ERROR:  relation "users" does not exist at character 15
STATEMENT:  SELECT * FROM users;
```
To avoid this SQL injection, use `$1` and `$2`. `sql` package makes sure to handle all of these for us. It's gonna make sure it escapes things that needs to be escaped and that that type of attack is not possible.

```go
name := "', ''); DROP TABLE users; --"
email := "vishal@govind.io"

_, err = db.Exec(`
	INSERT INTO users (name, email)
	VALUES ($1, $2);
`, name, email)

if err != nil {
	return panic(err)
}
```
Now, if we fetch users:
```sql
SELECT * FROM users;

-- output:
 id |             name              |      email       
----+-------------------------------+------------------
  1 | '', ''); DROP TABLE users; -- | vishal@govind.io
(1 row)
```

##### Acquiring New Record IDs
- One way is using `sql.Result` type returned by `sql.DB.Exec` method
	- but this is not supported by all database, for e.g. PostgreSQL doesn't support it
- Another way is using `RETURNING` clause in SQL with `sql.DB.QueryRow` (because we expect to return just single `Row`)
```go
name := "Vishal Govind"
email := "demo@user.com"
row := db.QueryRow(`
	INSERT INTO users (name, email)
	VALUES ($1, $2)
	RETURNING id, name;
`, name, email)

var id int
var rname string
err := row.Scan(&id, &rname)
if err != nil {
	panic(err)
}
fmt.Printf("User created. id = %v, name = %v", id, rname) 
```

#### Querying Table
##### Single Row - `sqlQueryRow(q string, args ...any) *Row`
```go
id := 6
row := db.QueryRow(`
	SELECT name, email
	FROM users
	WHERE id = $1;
`, id)
name := ""
email := ""
err = row.Scan(&name, &email)
if err != nil {
	panic(err)
}
fmt.Printf("User info: name=%s, email=%s\n", name, email)
```

##### Multiple Rows - `sql.Query(q string, args ...any) (*Rows, error)` 
- `defer rows.Close()` after fetching multiple rows
- use `rows.Next()` to loop over the rows
- `rows.Next()` returns false if there are any errors. So make sure to check for `rows.Err()` at the end of the `for rows.Next()` loop
```go
type Order struct {
	ID int
	UserID int
	Amount int
	Description string
}
var orders []Order
userID := 6
rows, err := db.Query(`
	SELECT id, amount, description
	FROM orders
	WHERE user_id = $1
`, userID)
if err != nil {
	panic(err)
}
// it is important to close to make sure this gets cleaned up and closed up whenever we're done with it.
defer rows.Close()

// make sure to look at every single row and try to parse them
// the way we range over the *Rows is a bit different, rather than doing something like `for range`, use rows.Next() iterator
for rows.Next() {
	var order Order
	order.UserID = userID
	err := rows.Scan(&order.ID, &order.Amount, &order.Description)
	if err != nil {
		panic(err)
	}
	orders = append(orders, order)
}
// check for error
err = rows.Err()
if err != nil {
	panic(err)
}

fmt.Println("Orders:", orders)
```