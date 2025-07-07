#go #go-template #html/template #contextual-encoding

- Step 1: Parse
- Step 2: Execute
### Example
```go
// ./hello.gohtml

<h1>Hello, {{.Name}}</h1>
{{.Age}}
{{.Meta.Visits}}
{{.Bio}}
```

```go
// ./main.go
package main

import (
	"html/template"
	"os"
)

type User struct {
	Name string
	Age int
	Meta UserMeta
}
type UserMeta struct {
	Visits int
}

func main() {
	t, err := template.ParseFiles("hello.gohtml")
	if err != nil {
		panic(err)
	}
	user := User{ Name: "Vishal", Age: 26, Meta: UserMeta{ Visits: 10 } }
	err = t.Execute(os.Stdout, user)
	if err != nil {
		panic(err)
	}
}

// ---------------------------
// Output:
// ---------------------------
// $ go run .
// <h1>Hello, Vishal</h1>
// 26
// 10
```

#### XSS
The html templating package also takes care of XSS #xss 
```go
// ./main.go
package main

import (
	"html/template"
	"os"
)

type User struct {
	Name string
	Age int
	Meta UserMeta
}
type UserMeta struct {
	Visits int
}

func main() {
	t, err := template.ParseFiles("hello.gohtml")
	if err != nil {
		panic(err)
	}
	user := User{ Name: "<script>alert("Haha, you have been hacked!");</script>", Age: 26, Meta: UserMeta{ Visits: 10 } }
	err = t.Execute(os.Stdout, user)
	if err != nil {
		panic(err)
	}
}

// ---------------------------
// Output:
// ---------------------------
// $ go run .
// <h1>Hello, &lt;script&gt;alert(&#34;Haha, you have been hacked!&#34;);&lt;/script&gt;</h1>%
// 26
// 10
```
- NOTE: XSS is not supported in the `text/template` package and only with `html/template` package

### What if we want to render actual HTML while using `html/template`
- Use `template.HTML` type
```go

// ./main.go
package main

import (
	"html/template"
	"os"
)

type User struct {
	Name string
	Bio template.HTML
	Age int
	Meta UserMeta
}
type UserMeta struct {
	Visits int
}

func main() {
	t, err := template.ParseFiles("hello.gohtml")
	if err != nil {
		panic(err)
	}
	user := User{ Name: "Vishal", Bio: "<script>alert("Haha, you have been hacked!");</script>" }
	err = t.Execute(os.Stdout, user)
	if err != nil {
		panic(err)
	}
}

// ---------------------------
// Output:
// ---------------------------
// $ go run .
// <h1>Hello, Vishal</h1>
// <script>alert("Haha, you have been hacked!");</script>
// 0
// 0
```

### Contextual Encoding by `html/template`
- HTML and JS encoding are done bit differently, thus encoding done is **contextual**  
```gohtml
<h1>Hello, {{.Name}}</h1>
{{.Bio}}

<script>
const user = {
	"name": {{.Name}},
	"bio": {{.Bio}},
}
console.log(user)
</script>
```

```go
// ./main.go
package main

import (
	"html/template"
	"os"
)

type User struct {
	Name string
	Bio string
}

func main() {
	t, err := template.ParseFiles("hello.gohtml")
	if err != nil {
		panic(err)
	}
	user := User{ Name: "Vishal", Bio: "<script>alert("Haha, you have been hacked!");</script>" }
	err = t.Execute(os.Stdout, user)
	if err != nil {
		panic(err)
	}
}
// ---------------------------
// Output: note that the js encoding within the <script> is different than what is in html encoding of the same string
// ---------------------------
// $ go run .
// <h1>Hello, Vishal</h1>
// &lt;script&gt;alert(&#34;Haha, you have been hacked!&#34;);&lt;/script&gt;

// <script>
// const user = {
//    "name": "Vishal",
//    "bio": "\u003cscript\u003ealert(\"Haha, you have been hacked!\");\u003c/script\u003e",
//    "age": 10, // notice age is printed without quotes (i.e. it is not "10") i.e. age is int 
// }
// console.log(user)  
// </script>
```
- ![[Contextual Encoding.png]]

### Using template
- `template.ParseFiles` returns error if there is anything wrong with the html files, like invoking undefined functions
- ideally we want to know about any template parsing errors even before starting up our server
- `template.Execute` returns an error when it is unable to access some of the referred fields in the template
	- like if the template refers to `{{.Name}}`, and the `data` passed to `template.Execute(w, data)` doesn't have `Name` field
	- but by that time, the template might have written some stuff to the writer `w` and while it encounters error, it pauses/stops further writing and returns error. Thus, `w` would have some of the stuff which executed correctly until the first error encountered
```go
// ./templates/home.gohtml

<html>
	<h1>Welcome To Golang</h1>
	<a href="/contacts">Contacts</a>
</html>
{{.Name}}
```

```go
// ./main.go
func homeHandler(w http.ResponseWriter, _ *http.Request) {
	path := filepath.Join("templates", "home.gohtml")
	t, err := template.ParseFiles(path)
	if err != nil {
		log.Printf("parsing template: %v", err)
		http.Error(w, "There was an error parsing template", http.StatusInternalServerError)
		return
	}
	
	err = t.Execute(w, "a string")
	if err != nil {
		http.Error(w, "There was an error executing template", http.StatusInternalServerError)
		return
	}
}
```
- here, the `w` would contain (both error message and correctly executed text so far) 
	- also the status code of this response would also not be changed to `http.StatusInternalServerError`
		- this is because, **once we start writing to a `http.ResponseWriter`, it sets the status code as `http.StatusOK` (200)**
		- and **status code can't be changed once it's been set**
```
# Welcome To Golang

Contacts There was an error executing template
```

##### How to get around this issue?
#io-Writer #strings-Builder #bytes-Buffer
- `bytes.Buffer` and `strings.Builder` implement `io.Writer`
- One way would be execute the entire template to a `bytes.Buffer` or `*strings.Builder` instead of directly to `http.ResponseWriter`, then if it executes successfully, then write it to `http.ResponseWriter`
```go
var buf = bytes.NewBufferString("")
// or var buf = &strings.Builder{}
err = t.Execute(buf, "name")
if err != nil {
	http.Error(w, "There was an error executing template", http.StatusInternalServerError)
	return
}
fmt.Fprint(w, buf.String())
```
