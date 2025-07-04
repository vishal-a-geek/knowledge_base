### Why Go
- Was invented by geniuses (Ken Thompson, Rob Pike, others at Google)
- Multi core
- efficient compile, run, ease of programming

### ASCI, Unicode, UTF-8
**ASCII**: 1 byte
**Unicode**: 4 bytes
	- use more than enough to account for every char
**UTF-8**: upto 4 bytes
- stores unicode as binary
- if a char needs 1 byte, that's all it will use
- 

### Raw string literal:
use \`\` (backticks)
```go
fmt.Println(`RAW String`)
```


### Type Conversions
- There's no type-casting thing in go
- `float32` is diff from `float64`
```
z := 42.0 // by default it's float64

var m float32 = 43.23 // specify other than float64

// z = m; // won't work as cannot use variable of 

type float32 as float64 value

z = float64(m); // this would work
```
