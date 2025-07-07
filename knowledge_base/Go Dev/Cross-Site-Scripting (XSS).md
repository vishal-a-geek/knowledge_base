#go #cross-site-scripting #sql-injection #xss

### [SQL Injection](https://xkcd.com/327/)
![[SQL - Injection.png]]

### XSS
```go

func homeHandler(w http.ResponseWriter, _ *http.Request) {
	bio := `<script>alert("Haha, you have been hacked!");</script>`
	fmt.Fprintf(w, `<h1>Welcome</h1><a href="/contacts">Contacts</a><p>Bio:%v</p>`, bio)
}
```
- The reason this happens is because the script tag in the bio is not being encoded
![[XSS.png]]
- If we want our browser to render this script tag instead of processing, we need to encode it.
- Thus, instead of the script tag, use `&lt;script&gt;alert(&quot;Hi!&quot;);&lt;/script&gt;`
![[XSS-2.png]]
![[XSS-3.png]]

### How to avoid XSS?
- Use `http/template` check [[Go Templating#XSS]]