#go #sentinel-errors 

- Indicate occurrence of an unique event
- Exported package level variables
```go
package psql

var (
	ErrNotFound = errors.New("")
)
```
- Example from [`database/sql`](https://pkg.go.dev/database/sql#ErrNoRows) package
```go
// database/sql package
package sql

var ErrNoRows = errors.New("sql: no rows in result set")

```
- To check specific error, we can do
```go
import (
	"database/sql"
)
if err == sql.ErrNoRows {
	// ... handle
}
```
