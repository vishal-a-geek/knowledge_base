#go #net/http #url-Values

#### `http.Request.ParseForm()` and `http.Request.PostForm`
- in [`http.Request`](https://pkg.go.dev/net/http#Request) there is a `PostForm` field which contains parsed form data from PATCH, POST or PUT body parameters
- `PostForm` field is only available after `ParseForm` is called
- [`ParseForm`](https://pkg.go.dev/net/http#Request.ParseForm) populates `r.Form` and `r.PostForm`
- `ParseForm` is idempotent, meaning, we can call it multiple times but it would have same result

To parse request
- Invoke `r.ParseForm()` and then use `url.Values` returned by `r.PostForm`

#### `url.Values`

```go
package url
// ...

type Values map[string][]string 
// the value of this map if of type []string because sometimes we might want to associate multiple values for same key
// for e.g while submitting multiple tags or something like that for a single key

func (v Values) Get(key string) string
// gets the first associated value with a given
// if we want all the multiple values associated
```
- Using `ParseForm`
```go

func (u Users) Create(w http.ResponseWriter, r *http.Request) {	
	err := r.ParseForm()
	if err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}
	fmt.Fprintf(w, "Email %s, Password %s", r.PostForm.Get("email"), r.PostForm.Get("password"))
}
```
#### `r.FormValue()`
[`FormValue`](https://pkg.go.dev/net/http#Request.FormValue) returns the first value for the named component of the query. The precedence order:

1. `application/x-www-form-urlencoded` form body (POST, PUT, PATCH only)
2. query parameters (always)
3. `multipart/form-data` form body (always)

FormValue calls [Request.ParseMultipartForm](https://pkg.go.dev/net/http#Request.ParseMultipartForm) and [Request.ParseForm](https://pkg.go.dev/net/http#Request.ParseForm) if necessary and **ignores any errors** returned by these functions. If key is not present, FormValue returns the empty string. To access multiple values of the same key, call ParseForm and then use Request.Form directly.
- Using `FormValue`
```go
 func (u Users) Create(w http.ResponseWriter, r *http.Request) {	
	fmt.Fprintf(w, "Email %s, Password %s", r.FormValue("email"), r.FormValue("password"))
}
```

NOTE:
There are packages that make this parsing easier.
- [`github.com/gorilla/schema`](https://github.com/gorilla/schema)
- 